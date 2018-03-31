---
title: "RSpec: Basic introduction"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Ruby
  - RSpec
  - Testing
---
During the Christmas period I've been playing with a unit test framework for [Chef](https://www.chef.io/) called [ChefSpec](https://github.com/chefspec/chefspec). 
It might seen strange to be looking at a unit test framework for a tool that essentially changes the state (or resources) of a server since the test itself will not perform the actual action.
<!--more-->

The value of ChefSpec is to determine what Chef would do, assert the values received by each resource, test the variances in behavior for different platforms or verify that the correct code path is run based on any feature toggles. The other benefits are that it will offer faster feedback and show any syntax errors in the Ruby code.

## Rspec
ChefSpec is built on top of another test framework called [RSpec](http://rspec.info/).

RSpec is [behavior driven development](https://en.wikipedia.org/wiki/Behavior-driven_development) framework specifically designed for Ruby.

For me, the best way to learn is take something you do know and add something you don't so before using ChefSpec it is good to get an understanding of RSpec.

This article assumes that you have Ruby installed and fimiliar with [installing gems](http://guides.rubygems.org/rubygems-basics/).

The first thing to do is to ensure RSpec is installed; from a command prompt type
```
rspec ---version
```

If RSpec is not recognised as a command or the Rspec version is less than version 3 then install the latest RSpec gem
```
gem install rspec
```

Retyping
```
rspec --version
```
Ought to display version 3 (or above) of RSpec.

### First test
By default when RSpec is run from the commandline it looks for files named with an underscore spec (_spec) suffix and an extension of .rb. The files should be located in the spec folder, you can, however, invoke RSpec with an explicity named file or folder path. 

An RSpec test consists of an example group and examples.

An example group is a collection of tests covering a particular behavior, the example group is declared with a method called describe(). The describe method accepts an abritrary number of parameters and a block; the parameter values represents a facet of the system that we want to test, the block contains code examples that are part of the example group.

Inside of the example group are examples which are declared with a method call it(). The it method accepts a parameter and a block, the parameter describes the test, the block contains any actions required to run the test.

If it sounds complicated, don't worry, it isn't.

To keep the first test simple we will create an example group. Open a command prompt, create a new file named add_spec.rb, the content will be a single Exmaple Group using the describe method
```
describe '2+2' do
end
```

Because the add_spec.rb file is NOT in a folder named spec Rspec needs to be run specifying a filename
```
rspec add_spec.rb
```
![01-describe-only]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/01-describe-only.jpg){: .align-center}

No examples found and no tests run; the next step is to create a pending example, there are a few ways to do this, the easiest is to use the it method without passing it a block.
```
describe '2+2' do
  it 'returns 4'
end
```
![02-pending-test]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/02-pending-test.jpg){: .align-center}

Note that if you DO pass a block to the it method but it is empty then the tests will not fail!
```
describe '2+2' do
  it 'returns 4' do
  end
end
```
![03-empty-it-block]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/03-empty-it-block.jpg){: .align-center}

Rspec has a large built-in expectation library allowing tests results to be asserted; the assertions are formed as part of the block supplied to the it method.
```
describe '2+2' do
  it 'returns 4' do
    expect(2+2).to eq(4)
  end
end
```
![04-first-test-expectation]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/04-first-test-expectation.jpg){: .align-center}

The tests run without fail but in this example we are not testing any production code and it is not obvious what tests have run.

## Formatting the RSpec results
To better see the groups and examples run Rspec can format the test result output using the minus f (-f) parameter. The results can be formatted in a document style, HTML or JSON.

Viewing the results in a document style produces the following
![05-rspec-document-format]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/05-rspec-document-format.jpg){: .align-center}

This is one of the great features of RSpec, test read like a conversations
> You: describe 2+2

> Someone else: returns 4

A better and more real world example is
> You: describe a new account

> Someone else: It should have a balance of zero

The latter example can be respresented in RSpec as 
```
describe 'A new account' do
  it 'should have a balance of zero' do
    account = Account.new
    expect(account.balance).to eq(0)
  end
end
```
![06-new-account-test]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/06-new-account-test.jpg){: .align-center}

## Testing an actual method
The RSpec test used earlier simply uses Ruby to add two numbers together; to keep the whole exercise simple our production code is going to reside in the add_spec.rb file. Modify the content of add_spec.rb file to include a method to add two numbers together and change our expectation from using Ruby to calling our method
```
# Add two numbers together
def add(number_one, number_two)
  number_one + number_two
end

describe '2+2' do
  it 'returns 4' do
    expect(add(2,2)).to eq(4)
  end
end
```
Re-run the test and they pass but there are not enough tests to put add method into production.

## Context and Multiple examples
The example group created in the prvious examples only contains one example. A better example group would be one that contains many examples to fully test our add method
```
# Add two numbers together
def add(number_one, number_two)
  number_one + number_two
end

describe 'add' do
  it 'returns 4' do
    expect(add(2,2)).to eq(4)
  end

  it 'returns -2' do
    expect(add(-6, 4)).to eq(-2)
  end
end
```
![07-multiple-add-tests]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/07-multiple-add-tests.jpg){: .align-center}

The tests have lost some expressiveness, going back to our conversation the tests are saying
>You: add

> Someone else: return 4

> Someone else: return -2

It doesn't really make sense, the conversation is more likely to go
> You: add

> You: When both numbers are positive

> You: 2+2

> Someone else: return 4

> You: When numbers are negative

> You: -6+4

> Someone else: return -2

RSpec has an additional method we can use to further describe our test; a method named context(). The context method is an alias for the describe method so they can be used interchangably but it is better to use describe for classes/subjects or methods under test and context to provide a little more detail about the relevance of the test. Context and describe can be nested almost idenfinately.

The description of the examples themselves can reflect the actual result or be made to read like english
```
# Add two numbers together
def add(number_one, number_two)
  number_one + number_two
end

describe 'add' do
  context 'when numbers are positive' do
    context 'two plus two' do
      it 'returns four' do
        expect(add(2,2)).to eq(4)
      end
    end
  end

  context 'when numbers are negative' do
    context 'minus six plus four' do
      it 'returns minus two' do
        expect(add(-6, 4)).to eq(-2)
      end
    end
  end
end
```
![08-multiple-add-tests-with-context]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/08-multiple-add-tests-with-context.jpg){: .align-center}

## File layout
There is nothing special that is required in order to run an RSpec test, it can be a file located anywhere as long as it contains
* Example groups (collection of tests grouped by common functionality)
* Examples (individual tests)

In order to use the tools effectively and as they were designed to be used a specific folder structure is required
* Code goes into a library folder
* RSpec tests go into a spec folder
* There is usually a spec_helper.rb file supplied that ensures all classes are loaded

Create a folder named lib, in that folder create a file named add.rb with the content
```
# Add two numbers together
def add(number_one, number_two)
  number_one + number_two
end
```

Create a folder named spec, in that folder create a file named spec_helper.rb with the content
```
require 'add'
```

In the spec folder create another folder nameed unit, in that folder create a file named add_spec.rb with the content
```
require 'spec_helper'

describe 'add' do
  context 'when numbers are positive' do
    context 'two plus two' do
      it 'returns four' do
        expect(add(2,2)).to eq(4)
      end
    end
  end

  context 'when numbers are negative' do
    context 'minus six plus four' do
      it 'returns minus two' do
        expect(add(-6, 4)).to eq(-2)
      end
    end
  end
end
```
![09-file-layout]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/09-file-layout.jpg){: .align-center}

Run RSpec!
```
rspec -fd
```
![10-just-rspec]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/10-just-rspec.jpg){: .align-center}

There is no need to specify running the add_spec.rb file because RSpec will automatically recurse the spec folder looking for files ending in _spec.rb

## Running a specific test
As you increase the number of tests you might find that you want to run a single test in insolation, Rspec permits that too.

Go to your _spec test file, locate the line number of the example you wish to run
![11-single-test-line-number]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/11-single-test-line-number.jpg){: .align-center}

it 'returns minus two' is on line 14, to run that Example specify the file and line number when running RSpec
```
rspec spec\unit\add_spec.rb:14
```
![12-single-test-line-number-result]({{ site.url }}{{ site.baseurl }}/assets/posts/2015-01-04-rspec-introduction/12-single-test-line-number-result.jpg){: .align-center}

## Summary
RSpec makes it very easy to start writing tests; the tests are expressive and simple to read.
