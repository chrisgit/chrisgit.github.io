---
title: "Nodejs: Mocha, basic introduction"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Nodejs
  - Testing
  - Mocha
---
For the past few days I've been making changes to a Nodejs project that is a plug module to a Nodejs web server.

Up until now the developer that has been doing the work up has the following workflow
- make changes to the code
- run a local copy of the Nodejs web server
<!--more-->
- go to the part of the system that uses the plug in
- manually test the code changes

There is no formal documentation of how to setup a local copy of the Nodejs web server and it has been a while since anyone has done it so if I decide to follow the same workflow it could be a few days of setup before I can make any changes to the codebase.

Given there is a certain amount of risk involved in setting up and running the server locally plus the length of time it takes to test the changes I've decided to reduce the feedback loop and improve confidence when refactoring the plugin by adding some unit tests.

## Mocha
The test framework that I have chosen to use is [Mocha](https://mochajs.org/) having used it before; as the tag line suggests Mocha does make testing simple and fun.

First thing to do is add Mocha to the project, we are using the [NPM package manager](https://www.npmjs.com/) rather than the sparkly new and speedy [Yarn package manager](https://yarnpkg.com/en/).

There is already a [packages.json file](https://docs.npmjs.com/files/package.json) included with the plugin making it easy to add Mocha by going to the root of the project and typing

```
npm install mocha --save-dev
```
Mocha only needs to be available for developers to run unit tests or for CI builds so we've used the [--save-dev parameter](https://docs.npmjs.com/cli/install).

NPM will update the packages.json file for us so that other developers or CI can retrieve the packages.

The plugin uses libraries that will only be available when it is loaded by the web server; in order to test our code we need to fake the loading of the packages and stub any calls to public methods; the Node package that I've chosen to do this is [mock-require](https://www.npmjs.com/package/mock-require). mock-require is only needed to support unit tests so it can be loaded as a developer dependency in the same way as Mocha.

```
npm install mock-require --save-dev
```

The other packages that I need for my unit tests are
- [should](https://www.npmjs.com/package/should) for test assertions
- [rewire](https://www.npmjs.com/package/rewire) for access to protected methods and variable
- [sinon](https://www.npmjs.com/package/sinon) for spies, stubs and mocks

```
npm install should --save-dev
npm install rewire --save-dev
npm install sinon --save-dev
```

## Writing the tests
[Visual Studio code](https://code.visualstudio.com/) is used to edit the web server plugins; to be able to run our unit tests we will add a simple  [launch configuration](https://code.visualstudio.com/docs/editor/debugging) file that can run a single test file.

To add a configuration click on the bug, open the drop down and select "Add Configuration" or click on the cog next to the drop down.

![01-visual-studio-add-launch-config]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-10-nodejs-mocha-introduction/01-visual-studio-add-launch-config.jpg){: .align-center}

The launch.json file will open in the editor, we need to add a new section inside the configurations array.

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Single File with mocha",
            "type": "node",
            "request": "launch",
            "program": "${workspaceRoot}/node_modules/mocha/bin/_mocha",
            "env": {
                "NODE_ENV": "test"
            },
            "stopOnEntry": false,
            "args": ["--no-timeouts", "${relativeFile}"],
            "cwd": "${workspaceRoot}"
        },
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}/lib/rtx-helper-unit.js"
        }
    ]
}
```

Now we are all set to run our unit tests; when you have unit tests click on the debug icon and selecting "Debug Single File with mocha" 

![02-visual-studio-debug]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-10-nodejs-mocha-introduction/02-visual-studio-debug.jpg){: .align-center}

For simplicity the unit tests will sit side by side in the same folder with the plugin code, the name of the test file is the same but instead of a .js extension it is given a .unit.js extension.

```
lib
   calculator.js
   calculator.unit.js
```

## Basic unit test
Before we dig in and start to use mock-require, rewire and sinon we will start with a basic unit test.

Start with a simple bit of code to add two numbers together, create a file named calculator.js
```JavaScript
exports.add = function(numberOne, numberTwo) {
    return numberOne + numberTwo;
};
```

Create the unit test file, name it calculator.unit.js. The unit test will
- load our calculator file 
- make a call to the add method
- assert that the result of calling the add method was correct

Mocha has two blocks that we are interested in
- describe allows us to give a test a human description or context
- it allows us to call the subject under test and assert the result

Both "describe" and "it" methods accept a callback function parameter, normally the callback is provided as an anonymous function.

Given our simple add method we can provide a simple test using describe and it.
```JavaScript
const calculator = require('./calculator.js')
describe('add', () => {
  it('returns correct result', () => {
      var result = calculator.add(5, 4);
      return result === 9;
  });
});
```
If you are test first developer you might want to write a series of tests and mark them as pending; you will do this if you want to check your code in to a CI system without breaking the build; Mocha allows pending tests by [omitting the function provided with the it method](https://mochajs.org/#pending-tests).

While the calculator.unit.js file is in the editor switch to the debugger and run the "Debug Single File with mocha".

The result of the tests will appear in the debug console

![03-visual-studio-debug-console]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-10-nodejs-mocha-introduction/03-visual-studio-debug-console.jpg){: .align-center}

## Making the tests more expressive
From the developer perspective the tests that you write should
- help describe the subject under test
- make it obvious what they are testing
- be simple and easy to maintain

The mocha test we wrote was easy to read but when you look at the debug console the assertion description is a bit vague. We are testing the add method and ensuring that the method provides the correct result, it doesn't tell us too much about the test, why we wrote it or what the parameters were. Mocha allows us to have nested describe blocks so that we can give the tests more context.
```JavaScript
describe('add', () => {
  describe('4 plus 5', () => {
    it('returns 9', () => {
        var result = calculator.add(5, 4);
        return result === 9;
    });
  });
});
```
Debug console now shows us more detail about the test

![04-visual-studio-debug-console-context]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-10-nodejs-mocha-introduction/04-visual-studio-debug-console-context.jpg){: .align-center}

The it assertion block has been given the description of "returns 9" rather than "should return 9" as "return 9" an explicit expectation.

The test uses a basic exact equal to perform the assertion but it is more common to use an assertion library. Most assertion libraries extend standard classes and provide a fluent API for you to join multiple statements together; they generally wrap standard operators with methods and provide functions to perform object equality and value equality.

The assertion library that I have used is should and my test can be updated to remove the exact equal statement with the equal method from the should framework.

```JavaScript
const calculator = require('./calculator.js')
const should = require('should');

describe('add', () => {
  describe('4 plus 5', () => {
    it('returns 9', () => {
        var result = calculator.add(5, 4);
        return result.should.eql(9);
    });
  });
});
```

## Test cases
Some test frameworks allow the tests to be data driven, this means your test code only needs to be written once but will accept parameters and expectations; if you have used something like NUnit you will likely be familiar with test cases.

Mocha does not provide the capability for writing test cases but you can write dynamic methods with JavaScript; we create an array of objects that can then be used to drive the tests.
```JavaScript
describe('add', () => {
  var tests = [
      {numberOne: -1, numberTwo: -3,  expected: -4, description: 'when numbers are negative'},
      {numberOne: 4,  numberTwo: 5,  expected: 9, description: 'when both numbers positive'},
      {numberOne: 3, numberTwo: 0,  expected: 3, description: 'when one number is zero'},
    ];

  tests.forEach(function(test) {
    describe(`${test.description}: ${test.numberOne} + ${test.numberTwo}`, () => {
      it('returns ' + test.expected, () => {
        var result = calculator.add(test.numberOne, test.numberTwo);
        return result.should.eql(test.expected);
      });
    });
  });
});
```
The resultant output is

![05-visual-studio-debug-console-testcases]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-10-nodejs-mocha-introduction/05-visual-studio-debug-console-testcases.jpg){: .align-center}

The test code avoids duplicating the same test for different values but gain some added complexity in the test code and less readability.

With the test framework in place I now need to disseminate the plugin and start to test it .
