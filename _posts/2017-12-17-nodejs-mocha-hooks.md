---
title: "Nodejs: Mocha, hooks"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Nodejs
  - Testing
  - Mocha
---
For the past few days I've been making changes to a Nodejs project that is a plug module to a Nodejs web server. The plugin is tested in a runtime environment using a local version of the Nodejs server; which has a specific set of installation instructions.

The first task I undertook was to add unit tests to the plugin, apart from adding confidence in any refactoring exercise it ought to give me faster feedback.<!--more--> My choice of the testing framework is Mocha.

Like most test frameworks Mocha has provision for test setup and teardown by means of before and after hooks, this post will help show when the hooks are fired.

## Order of execution
Mocha contains a "describe" method for you to group your tests with a context and an "it" method where you can make your assertions.

The example below contains two simple describe blocks.

```javascript
// Order of execution
describe('example block one', function(){
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one nested', function(){
        it('test 1-2', () => {});
        describe('example block one nested again', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    it('test 2-1', () => {});
    describe('example block two nested', function(){
        it('test 2-2', () => {});
    });
});
```

The output is linear with tests being executed in order.

![01-order-of-execution]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/01-order-of-execution.jpg){: .align-center}

## before and after hooks
The before hook will execute before any tests in the block are executed.
The after hook will execute after all of the test in the block have been executed.

Below is a basic example, the hooks are placed at the top of the file.

```javascript
// Using the before and after hooks
before( () => { console.log('before'); });
after( () => { console.log('after'); });

describe('example block one', function(){
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one (nested)', function(){
        it('test 1-2', () => {});
        describe('example block one (nested again)', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    it('test 2-1', () => {});
    describe('example block two (nested)', function(){
        it('test 2-2', () => {});
    });
});
```

The output is as expected; before hook is the first thing run, the after hook is the last thing run.

![02-before-and-after]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/02-before-and-after.jpg){: .align-center}

The before and after hooks can be nested and placed inside any describe block.

```javascript
// Using the before and after hooks
before( () => { console.log('before'); });
after( () => { console.log('after'); });

describe('example block one', function(){
    before( () => { console.log('  before (example block one)'); });
    after( () => { console.log('  after (example block one)'); });
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one (nested)', function(){
        it('test 1-2', () => {});
        describe('example block one (nested again)', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    before( () => { console.log('  before (example block two)'); });
    after( () => { console.log('  after (example block two'); });
    it('test 2-1', () => {});
    describe('example block two (nested)', function(){
        it('test 2-2', () => {});
    });
});
```

The outer most before is the first hook run, the outer most after is the last hook run.

![03-before-and-after-nested]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/03-before-and-after-nested.jpg){: .align-center}

## beforeEach and afterEach hooks
The beforeEach hook will execute before each of the tests in the block are executed.
The afterEach hook will execute after each of the test in the block have been executed.

Below is a basic example, the hooks are placed at the top of the file.

```javascript
// Using the beforeEach and after hooksEach
beforeEach( () => { console.log('beforeEach'); });
afterEach( () => { console.log('afterEach'); });

describe('example block one', function(){
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one (nested)', function(){
        it('test 1-2', () => {});
        describe('example block one (nested again)', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    it('test 2-1', () => {});
    describe('example block two (nested)', function(){
        it('test 2-2', () => {});
    });
});
```

![04-before-each-and-after-each]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/04-before-each-and-after-each.jpg){: .align-center}

The beforeEach and afterEach can be nested.

```javascript
// Using the before and after hooks
beforeEach( () => { console.log('beforeEach'); });
afterEach( () => { console.log('afterEach'); });

describe('example block one', function(){
    beforeEach( () => { console.log('  beforeEach (example block one)'); });
    afterEach( () => { console.log('  afterEach (example block one)'); });
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one (nested)', function(){
        it('test 1-2', () => {});
        describe('example block one (nested again)', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    beforeEach( () => { console.log('  beforeEach (example block two)'); });
    afterEach( () => { console.log('  afterEach (example block two'); });
    it('test 2-1', () => {});
    describe('example block two (nested)', function(){
        it('test 2-2', () => {});
    });
});
```

As with the before and after hooks the outer most beforeEach is run first and the outermost afterEach is run last.

![05-before-each-and-after-each-nested]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/05-before-each-and-after-each-nested.jpg){: .align-center}

## Mixing both types of hooks
Mixing the hooks will have no affect on the order of execution 

```javascript
// Using the before and after hooks
before( () => { console.log('before'); });
after( () => { console.log('after'); });
beforeEach( () => { console.log('beforeEach'); });
afterEach( () => { console.log('afterEach'); });

describe('example block one', function(){
    it('test 1-1', () => {});
    it('pending test 1-1');
    describe('example block one (nested)', function(){
        it('test 1-2', () => {});
        describe('example block one (nested again)', function(){
            it('test 1-3', () => {});
        });
    });
});

describe('example block two', function(){
    it('test 2-1', () => {});
    describe('example block two (nested)', function(){
        it('test 2-2', () => {});
    });
});
```

- before hook executes first
- after hook executes last
- beforeEach executes before each test in the block
- afterEach executes after each test in the block 

![06-mixed-hooks]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/06-mixed-hooks.jpg){: .align-center}

The hooks can be nested and appear in any describe block but don't go to crazy!

```javascript
// Nested before and after hooks nested in a block
before( () => { console.log('before'); });
after( () => { console.log('after'); });
beforeEach( () => { console.log('beforeEach'); });
afterEach( () => { console.log('afterEach'); });

describe('example block one', function(){
    before( () => { console.log('  before (nested)'); });
    after( () => { console.log('  after (nested)'); });
    beforeEach( () => { console.log('  beforeEach (nested)'); });
    afterEach( () => { console.log('  afterEach (nested)'); });

    it('test 1-1', () => {});
    it('pending test 1-1');

    describe('example block one nested', function(){
        it('test 1-2', () => {});
    });
});
```

Remeber before will always appear before a beforeEach and after will always appear at the end of the tests.

![07-mixed-hooks-and-nesting]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/07-mixed-hooks-and-nesting.jpg){: .align-center}

## Scope
Using begin and end does not affect the variable scope.

```javascript
// Doesn't affect scope
const should = require('should');
var testSetup = "Testing";

before(() => {
    testSetup = "Hello World";
});

after(() => {
    testSetup = "Testing";
});

describe('example block one', function(){
    it('testSetup is Hello World', () => {
        testSetup.should.eql("Hello World");
    });

    var anotherTestContext = "Goodbye World";
    it('testSetup is Goodbye World', () => {
        anotherTestContext.should.eql("Goodbye World");
    });
});
```

It is after all just JavaScript.

![08-scope]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-12-17-nodejs-mocha-hooks/08-scope.jpg){: .align-center}

The examples are contrived, in reality you will probably want to use your before/beforeEach as test setup to create objects and after/afterEach as test teardown to destroy objects; just as you would with a lot of other test frameworks.
