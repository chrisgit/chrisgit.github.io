---
title: "Nodejs: Mocha, refactoring"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Nodejs
  - Testing
  - Mocha
---

The code base I am working on is not fully tested and code coverage is low; before refactoring some of the methods I'd like some assurances that I will not break existing functionality.
<!--more-->

To be fair the tests are closer to integration tests than unit tests; the program is to convert data via an [Extract, Transform and Load (ETL)](https://en.wikipedia.org/wiki/Extract,_transform,_load). The documents passed in to the ETL do not have all the possible outcomes for transforming the data from one value to another. It is these transformation methods that have the lowest test coverage as they are written using multiple branching statements similar to the code below
```javascript
function convertFuelType(legacyFuelType) {
    var fueltype = '';
    switch (legacyFuelType) {
    case 1:
        fueltype = 'D';
        break;
    case 2:
        fueltype = 'E';
        break;
    case 3:
        fueltype = 'H';
        break;
    default:
        fueltype = 'A';
        break;
    }
    return fueltype;
}
```

The first thing to do before changing any code is to test what is currently there given our understanding of the code. The example presented here is pretty simple but I have come across methods that are well over a hundred [lines of code (LOC)](https://en.wikipedia.org/wiki/Source_lines_of_code); although those types of methods need different refactoring techniques.

In this example we have five possible outcomes, rather than write five test methods I've generated tests based on input and output data; it does make the tests less readable but driving the tests from data adds an element of flexibility. The test data contains the legacy value, the value we expect the method to return and a small description that isn't necessary for the test but may aid validating the result. 
```javascript
describe('Fuel types', () => {
    var tests = [
        {legacy: 1,       expected: 'D', Description: 'Diesel'},
        {legacy: 2,       expected: 'E', Description: 'Electric'},
        {legacy: 3,       expected: 'H', Description: 'Hybrid'},
        {legacy: 4,       expected: 'A', Description: 'All others'},
        {legacy: 99,      expected: 'A', Description: 'All others'}
        ];

    tests.forEach(function(test) {
        it('Fuel type of ' + test.legacy + ' returns code ' + test.expected, () => {
            return mapper.convertFuelType(test.legacy).should.eql(test.expected);
        });
    });
});
```

## Refactoring
Now that we have 100% code coverage is there any need to refactor the code? At nineteen lines long this method is not difficult to read but other areas of the code have more complex mapping with several values mapping to one return value, in the example below the incoming value of 2 and 3 return a value of 2, the entire method has a case statement mapping thirty values
```javascript
    var freq = 4;
    switch (val) {
    case 1:
        freq = 1;
        break;
    case 2:
    case 3:
        freq = 2;
        break;
```
When the case has lots of outcomes it is harder to determine what the result will be, particularly if there is some pre or post processing in the method.

To simplify the maintenance and reduce complexity for many of these conversions I've moved the mapping to an object (the use of freeze seems to have divided opinions in the Node/JavaScript community but I've used it here).
```javascript
var ApplicationConstants = Object.freeze({
    fuelTypes: Object.freeze({
        1: 'D',
        2: 'E',
        3: 'H',
        'default': 'A'
    })
});

function convertFuelType(legacyFuelType) {
    return ApplicationConstants.fuelTypes[legacyFuelType] || ApplicationConstants.fuelTypes.default;
}
```

Of course now we have total confidence that our changes work because we had the tests.

![01-refactoring-test-results]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-22-nodejs-mocha-refactoring/01-refactoring-test-results.jpg){: .align-center}

The downside is that the code only contains two paths which means only two tests are required to give us 100% code coverage; great but that would not prove all possible outcomes so as the mapping object expands so really should the unit tests.
```javascript
describe('Fuel types', () => {
    var tests = [
        {legacy: 1,       expected: 'D', Description: 'Diesel'},
        {legacy: 99,      expected: 'A', Description: 'All others'}
    ];

    tests.forEach(function(test) {
        it('Fuel type of ' + test.legacy + ' returns code ' + test.expected, () => {
            return mapper.convertFuelType(test.legacy).should.eql(test.expected);
        });
    });
});

```

![02-refactoring-test-code-coverage]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-22-nodejs-mocha-refactoring/02-refactoring-test-code-coverage.jpg){: .align-center}

Ultimately having 100% code coverage without testing all outcomes is the downside of data driven lookups.
