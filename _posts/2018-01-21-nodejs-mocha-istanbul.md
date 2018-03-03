---
title: "Nodejs: Mocha, instanbul"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Nodejs
  - Testing
  - Mocha
---

Retrospectively adding unit tests to a project can be a time consuming exercise; for some it is easier to take the existing result of the method and match that to the expectation of the test, particularly if they do not know what the intent of the original development, besides, system already works right?

Having introduced the Mocha testing framework to some of our Node components I wanted to cover the largest possible surface area <!--more--> to maximize my efforts so wrote largely integration style tests stubbing at the system boundaries, such as calls to external services.

Confidence in refacoring the code is higher than it was but what I'd like to know is how much of the code base in being tested and what could I do to improve that.

## Code coverage
In simple terms code coverage measures what percentage of code has been exercised by a test suite.

To give an example imagine we have a program that returns the number that is passed as a parameter
```javascript
exports.number = function(number) {
    return number;
};
```
The code only has one code path, a single test will cover all code paths. Given the following simple test we have 100% code coverage.
```
const should = require('should');
const fizzBuzz = require('./fizzbuzz')

describe('Fizzbuzz', () => {
  it('returns correct result', () => {
      return fizzBuzz.number(4).should.eql(4);
  });
});
```

Our really cool program has a new requirement, if the number is divisble by three then we want to return the word "fizz" so we add a branching condition to handle this.

```javascript
exports.number = function(number) {
    if (number % 3 == 0) {
        return 'fizz'
    }
    return number;
};
```

Our code now has two paths, one for numbers divisible by three and one for numbers that are not divisible by three. With our one test we have 50% code coverage as we have only tested one path. The tests need to be updated, a minumum of two test are required to give us 100% code coverage.

```javascript
const should = require('should');
const fizzBuzz = require('./fizzbuzz')

describe('Fizzbuzz', () => {
  it('returns number', () => {
      return fizzBuzz.number(4).should.eql(4);
  });
  it('returns fizz', () => {
    return fizzBuzz.number(3).should.eql('fizz');
  });
});
```

Surely there must be some tooling to help us with our code coverage?

## Istanbul
[Istanbul](https://www.npmjs.com/package/istanbul) is a Javascript code coverage tool written in JavaScript and plays very well with Mocha.

There are a couple of ways to install istanbul with npm.

### Installing globally
The instructions for installing istanbul are very simple
```
npm install -g istanbul
```
Installing istanbul will make it available to all of your Node projects.
NB: After installing istanbul globally restart your command window.

If Mocha is installed globally you can just call istanbul with the cover parameter and specify the test files (or glob)
```
istanbul cover lib/fizzbuzz.unit.js
```

If Mocha is installed locally to your project but istanbul is installed globally you will have to specify the path to Mocha in order to run unit tests
```
istanbul cover ./node_modules/mocha/bin/_mocha lib/fizzbuzz.unit.js
```

If you do not specify the path to Mocha the test will fail to load as the Mocha references are unavailable, you'll get something like the output below
```
ReferenceError: describe is not defined
    at Object.<anonymous> (/tmp/nodejs_demos/lib/fizzbuzz.unit.js:9:183)
    at Module._compile (module.js:643:30)
    at Object.Module._extensions.(anonymous function) [as .js] (/home/chris/.nvm/versions/node/v8.9.4/lib/node_modules/istanbul/lib/hook.js:107:24)
    at Module.load (module.js:556:32)
    at tryModuleLoad (module.js:499:12)
    at Function.Module._load (module.js:491:3)
    at Function.Module.runMain (module.js:684:10)
    at runFn (/home/chris/.nvm/versions/node/v8.9.4/lib/node_modules/istanbul/lib/command/common/run-with-cover.js:122:16)
    at /home/chris/.nvm/versions/node/v8.9.4/lib/node_modules/istanbul/lib/command/common/run-with-cover.js:251:17
    at /home/chris/.nvm/versions/node/v8.9.4/lib/node_modules/istanbul/lib/util/file-matcher.js:68:16
```

When istanbul is installed globally it is easier to run it via npm scripts, edit the package.json file to call istanbul, the extract below is using a globally installed istanbul and local Mocha
```
"scripts": {
    "cover": "istanbul cover ./node_modules/mocha/bin/_mocha -- --reporter spec --check-leaks \"**/*.unit.js\"",
    "postcover": "xdg-open ./.cov/index.html"
}
```

The cover script will run instanbul which in turn runs the Mocha tests maching the glob *.unit.js.
The postcover script runs automatically after the cover script; here postcover opens the generated index.html file. The command xdg-open is the command for Linux, when using Windows xdg-open can be replaced with 
```
cmd /c start index.html
```

The script from package.json can be run with npm as follows
```
npm run-script cover
```

Alternatively there is a runner for istanbul called [nyc](https://www.npmjs.com/package/nyc) which can be placed in your npm scripts section.

### Installing locally
Install istanbul locally to your project using npm with --save-dev or --save
```
npm install istanbul --save-dev
```

When istalbul is installed locally to the project you will have to call "cli.js", if Mocha is installed locally the you will need to refer to the projects version of Mocha; to run our single test
```
./node_modules/istanbul/lib/cli.js cover --x lib/fizzbuzz.unit.js --report html --dir ./.cov ./node_modules/mocha/bin/_mocha lib/fizzbuzz.unit.js
```

To invoke multiple test files use a glob.
```
 ./node_modules/istanbul/lib/cli.js cover --x "**/*.unit.js" --report html --dir ./.cov ./node_modules/mocha/bin/_mocha -- --reporter spec--check-leaks "**/*.unit.js"
```

Rather than repeat the command multiple times it is easier to run it as an npm script as a test script or a seperate coverage section to package.json
```
"scripts": {
    "cover": "./node_modules/istanbul/lib/cli.js cover --x \"**/*.unit.js\" --report html --dir ./.cov ./node_modules/mocha/bin/_mocha -- --reporter spec --check-leaks \"**/*.unit.js\"",
    "postcover": "xdg-open ./.cov/index.html"
}
```
NB: As the command is a string in a JSON file the double quotes are escaped

The cover script will run istanbul which in turn runs the Mocha tests maching the glob *.unit.js.
As with globally installed istanbul the script from package.json can be run with npm as follows
```
npm run-script cover
```

## Making use of our coverage tool
Now we have some code coverage tooling, lets see it in action. We will make another change to our program, if a number is divisible by 5 we will return the word "buzz".
```javascript
exports.number = function(number) {
    if (number % 3 == 0) {
        return 'fizz'
    }
    if (number % 5 == 0) {
        return 'buzz'
    }
    return number;
};
```
Leave the tests as they are and run the code coverage, the settings we have specified means istanbul will generate us a nice HTML file to browse.

The inital level of the report gives us a nice summary of our coverage
![01-instanbul-html-top-level]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-21-nodejs-mocha-instanbul/01-instanbul-html-top-level.jpg){: .align-center}

Drilling down we can see the coverage for all of the files in the project; just one file for now
![02-instanbul-html-fizzbuzz]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-21-nodejs-mocha-instanbul/02-instanbul-html-fizzbuzz.jpg){: .align-center}

Selecting a file will give the actual coverage
![03-instanbul-html-fizzbuzz-path-not-taken]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-21-nodejs-mocha-instanbul/03-instanbul-html-fizzbuzz-path-not-taken.jpg){: .align-center}

It doesn't look like we have 100% coverage, I think we need to write some more tests!
