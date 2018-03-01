---
title: "Nodejs: Mocha and mock-require"
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

[mock-require](https://www.npmjs.com/package/mock-require) is a library that allows us to bypass the normal loading mechanism for Node, why would you want to do that? Read on ...

## The problem
Lets start with a basic calculator file

```javascript
exports.add = function(numberOne, numberTwo) {
    return numberOne + numberTwo;
};
```

with some a basic unit test
```javascript
const should = require('should');
const calculator = require('./calculator.js')

describe('add', () => {
  describe('4 plus 5', () => {
    it('returns 9', () => {
        var result = calculator.add(5, 4);
        return result.should.eql(9);
    });
  });
});
```

There is a new requiement the functionality needs to change to include a function library from a Nodejs server that is only available at runtime; adding the library for development poses some problems.

We make the change to the code use our library
```javascript
const commonLib = require('@company/lib/core')

exports.add = function(numberOne, numberTwo) {
    return numberOne + numberTwo;
};
```

but running the unit tests result in an error

![01-module-load-failure]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-07-nodejs-mocha-mock-require/01-module-load-failure.jpg){: .align-center}

For now What we actually want to do is ignore the loading of any unavailable libraries to allow our tests to run.

## Fake loading the libraries
To spoof the loading of the library I have used [mock-require](https://www.npmjs.com/package/mock-require), note the dash between mock and require, as an aside there are other libraries that perform the same function.

The test file needs to change to include mock-require; mock-require then needs to be told to spoof the loading of our library.

If you do not configure mock-require libraries are loaded in the normal way so your tests can have a mixture of libraries that are actually loaded and libraries that are spoofed. Any libraries that are handled with mock-require need to be declared before you load the subject under test, in this case our calculator.js file.

After adding mock-require to the project with NPM
```
npm install mock-require --save-dev
```

We can amend our test

```javascript
const should = require('should');
const mockRequire = require('mock-require');
mockRequire('@company/lib/core');
const calculator = require('./calculator.js')

describe('add', () => {
  describe('4 plus 5', () => {
    it('returns 9', () => {
        var result = calculator.add(5, 4);
        return result.should.eql(9);
    });
  });
});
```

Now our test file runs and our tests pass.

### Mocking out functions
Sometime a specific function is loaded into a constant or variable such as

```javascript
const tracerJS = require('@company/lib/core').getTracer("jsRunner");;

exports.add = function(numberOne, numberTwo) {
    return numberOne + numberTwo;
};
```

If we spoof the loading of the module with mock-require the test will fail as getTracer will not be available

![02-module-load-failure-get-tracer]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-07-nodejs-mocha-mock-require/02-module-load-failure-get-tracer.jpg){: .align-center}

Fortunately mock-require will allow us to create a stub method to call into

```javascript
const should = require('should');
const mockRequire = require('mock-require');
mockRequire('@company/lib/core', {
  getTracer: function() {
      console.log('tracer output');
  }
});
const calculator = require('./calculator.js')

describe('add', () => {
  describe('4 plus 5', () => {
    it('returns 9', () => {
        var result = calculator.add(5, 4);
        return result.should.eql(9);
    });
  });
});
```

Now our test file runs and our tests pass.

![03-passing-tests]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-07-nodejs-mocha-mock-require/03-passing-tests.jpg){: .align-center}

