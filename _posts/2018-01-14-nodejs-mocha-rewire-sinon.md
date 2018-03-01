---
title: "Nodejs: Mocha, rewire and sinon"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Nodejs
  - Testing
  - Mocha
---
Late last year I'd been making changes to a Nodejs project that is a plug module to a Nodejs web server. The plugin is tested in a runtime environment using a local version of the Nodejs server; which has a specific set of installation instructions.

The plugin requires libraries that are only available when it is running in the context of the Nodejs Web server but my goal is to give the plugin a full suite of unit tests.
<!--more-->

[rewire](https://www.npmjs.com/package/rewire) is a library that adds a special setter and getter to modules so you can modify their behaviour for better unit testing.

[sinon](https://www.npmjs.com/package/sinon) is a library that enables our tests to use spies, stubs and mocks.

There is a lot of synergy between these two libraries so hopefully we can see that synergy in action.

## Demonstration
Let's say we have a service that will upload a file to a file store, we expose a method called zipAndUploadFile that takes a filename and file as parameters. Our program will zip the file and send it to the file store. The awesome code is in development and here is what we have so far

```javascript
var zip = require('adm-zip');

function zipFile(filename, file) {
	var dataBuffer = new Buffer(JSON.stringify(file), 'utf-8');
	var zipper = new zip();
	zipper.addFile(filename + '.json', dataBuffer);
	return zipper.toBuffer();
}

exports.zipAndUploadFile = function(filename, file) {
	var zipped = zipFile(filename, file);

    // The rest of the code here...
	var response = {
		"statusCode": 200,
		"message": "All good here"
	}
    return response;
}
```

For this test we aren't bothered about making sure that the Zip function works but we do want to make sure the upload works. In the root of the project we add the libraries that are required for this example
```
npm install adm-zip --save-dev
npm install mocha --save-dev
npm install should --save-dev
npm install sinon --save-dev
npm install rewire --save-dev
```

We are using rewire so that we can override the Zip function.
rewire is used to load our file under test instead of using Node's require.
```javascript
const rewire = require('rewire');
const fileService = rewire('./fileservice_justzip.js');
```
Create a Sinon stub that returns an empty object.
```javascript
var zipFile = sinon.stub();
zipFile.returns({});
```
rewire exposes getters and setters for each of our methods, we use a setter to replace the zipFile method with the Sinon stub.
```javascript
var unsetZipFile = fileService.__set__('zipFile', zipFile);
``` 

The entire test is below
```javascript
const should = require('should');

const sinon = require('sinon');
const rewire = require('rewire');
const fileService = rewire('./fileservice.js');

describe('zipAndUploadFile', () => {
    before(function() {
        var zipFile = sinon.stub();
        zipFile.returns({});
        var unsetZipFile = fileService.__set__('zipFile', zipFile);
    });

    describe('when zipping and uploading file', () => {
        it('returns 200', () => {
            var result = fileService.zipAndUploadFile('dummy.txt');
            return result.statusCode.should.eql(200) && result.message.should.eql('All good here');
        });
    });
});
```

Running the test gives us a 200.

![01-zip-file-override-test-results]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-14-nodejs-mocha-rewire-sinon/01-zip-file-override-test-results.jpg){: .align-center}

## Sinon stubbing multiple calls
As our service progesses we add some checks
- does the file already exist in the file store?
- yes: delete the file from the file store
- upload the file and check for errors

Our program communicates to the file store via an API accessible with http calls, we add a httpHeader and httpRequest method
```javascript
function httpHeader() {
    return {
		"Content-Type": "application/json",
		"Accept": "application/json",
		"x-application": "coolio",
		"x-user-id": "barney"
	};
}

function httpRequest(method, route, header, body, encoding) {
	var response = {};
	// Perform the httpRequest ...
	return response;
}
```

The costs of our additional behavior is that we need to make multiple calls to the file stores API, we have methods for checking the existance of the file, deleting the file and uploading the file.
```javascript
function checkFileExists(path, filename) {
	return httpRequest('GET', path, httpHeader());
}

function deleteFile(path, filename) {
	return httpRequest('DELETE', path, httpHeader());
}

function uploadFile(path, file) {
	var body = {
		"description": "string",
		"mimeType": "application/zip",
		"size": file.length
	}
	return httpRequest('POST', path, httpHeader(), JSON.stringify(body));
}
```

The entire awesome program now looks like this
```javascript
var zip = require('adm-zip');

function parseError(result) {
	var resp = {
		"statusCode": result.header.statusCode,
		"message": result.header.message
	}
	return resp;
}

function zipFile(_, fileName, file) {
	var dataBuffer = new Buffer(JSON.stringify(file), 'utf-8');
	var zipper = new zip();
	zipper.addFile(fileName + '.json', dataBuffer);
	return zipper.toBuffer();
}

exports.zipAndUploadFile = function(filename) {
	var path = 'https://somebucket/';
	var zipped = zipFile(filename);

	var response = checkFileExists(path, filename);
	if (response.header.statusCode === 204) {
		response = deleteFile(path, filename);
	}

	response = uploadFile(path, zipped);
	if (response.header.statusCode != 200) {
		return parseError(response);
	}

	response = {
		"statusCode": 200,
		"message": "All good here"
	}	
	return response;
}

function checkFileExists(path, filename) {
	return httpRequest('GET', path, httpHeader());
}

function deleteFile(path, filename) {
	return httpRequest('DELETE', path, httpHeader());
}

function uploadFile(path, file) {
	var body = {
		"description": "string",
		"mimeType": "application/zip",
		"size": file.length
	}
	return httpRequest('POST', path, httpHeader(), JSON.stringify(body));
}

function httpHeader() {
    return {
		"Content-Type": "application/json",
		"Accept": "application/json",
		"x-application": "coolio",
		"x-user-id": "barney"
	};
}

function httpRequest(method, route, header, body, encoding) {
	var response = {};
	// Perform the httpRequest ...
	return response;
}
```

We need to fake up to three responses to the httpRequest method. Sinon allows us to specify canned responses for each call to the stub using the onCall method, we can again use rewire to override the httpRequest method.
```javascript
var requestStatusResponse = sinon.stub();
requestStatusResponse.onCall(0).returns(checkFileExistsResponse());
requestStatusResponse.onCall(1).returns(deleteFileResponse());
requestStatusResponse.onCall(2).returns(fileUploadResponse());
var unsetHttpRequest = fileService.__set__('httpRequest', requestStatusResponse);
```

The entire test file now looks like this
```javascript
const should = require('should');

const sinon = require('sinon');
const rewire = require('rewire');
const fileService = rewire('./fileservice.js');

describe('uploadFile', () => {
    before(function() {
        var zipFile = sinon.stub();
        zipFile.returns({});
        var unsetZipFile = fileService.__set__('zipFile', zipFile);

        var requestStatusResponse = sinon.stub();
        requestStatusResponse.onCall(0).returns(checkFileExistsResponse());
        requestStatusResponse.onCall(1).returns(deleteFileResponse());
        requestStatusResponse.onCall(2).returns(fileUploadResponse());
        var unsetHttpRequest = fileService.__set__('httpRequest', requestStatusResponse);
    });

    describe('when zipping and uploading file', () => {
        it('returns 401', () => {
            var result = fileService.zipAndUploadFile('dummy.txt');
            return result.statusCode.should.eql(401) && result.message.should.eql('Unauthorised');
        });
    });
});

function checkFileExistsResponse() {
    return {
      header: { statusCode: 204 },
      body: {}
    }
}

function deleteFileResponse() {
    return {
      header: { statusCode: 200 },
      body: {}
    }
}

function fileUploadResponse() {
    return {
      header: { 
          statusCode: 401,
          message: "Unauthorised"
      },
      body: {}
    }
}
```

Running the test we see that we have proven that a 401 will produce an error and fail the upload

![02-zip-file-sinon-multiple-stub]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-14-nodejs-mocha-rewire-sinon/02-zip-file-sinon-multiple-stub.jpg){: .align-center}

Loving Nodejs, Mocha and supporting libraries, it is simple and fun!