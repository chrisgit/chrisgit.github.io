---
title: "MSTest: Running tests from the command line"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - mstest
---

As a developer I find one of the hardest things to get right is unit testing.

In order to produce the best unit tests I employ a combination of experimentation, knowledge and previous experience, sure you have to test happy cases, edge cases and attempt to cover all possible outcomes but how do you write enough tests to do that without hindering refactoring?

<!--more-->

For C# my passage into unit tests was influenced by my peers and books such as xUnit [Test Patterns: Refactoring Test Code](http://xunitpatterns.com/) and [The Art of Unit Testing](https://www.manning.com/books/the-art-of-unit-testing). For a long time I used [nUnit](http://nunit.org/) as my test framework of choice along with [Rhino mocks](https://www.hibernatingrhinos.com/oss/rhino-mocks) for my stubs and mocks. There was a brief excursion into the world of BDD with [Specflow](http://specflow.org/) but it was never really embraced by the department as something to move forward with.

It is fair to say I was reasonably ignorant of other test styles and frameworks until I started to use Ruby; it was Ruby, RSpec and Cucumber that really opened my eyes to good unit test practices. After writing my first RSpec test I was blown away with how quick it was to write and simple to read back. The long test method names that I had used in C# were now nested descriptions contained in RSpec's describe or Context methods. The Ruby community taught me to make the assertion descriptions positive, simple things, not "it should return X" but rather "it returns X". RSpec is very flexible and expressive, tests and bahaviors can easily be shared reducing code, it is obvious what the Class or Subject under test is, simple hooks that can also be nested, testing with RSpec is really fun.

After using RSpec I've been attracted to similar test frameworks for other languages such as [Mocha](https://mochajs.org/) for JavaScript/Node and [Ginko](https://onsi.github.io/ginkgo/) for Go.

## MSTest
This blog entry is not about unit tests or techniques but rather the use of a specific framework, MSTest, in fact the whole point of looking at test frameworks is that I have to submit a coding exercise for review, I could fall back to what I know and use nUnit or choose a framework similar on style to RSpec such as [NSpec](http://nspec.org/). After much thought I decided to go with Microsoft's own test solution MSTest; having a quick read of the documentation it doesn't seem that different from nUnit and it's all built in go to go, or so I thought.

The test class and methods for MSTest need to be attributed in the same way as nUnit, the names of the attributes themselves are the different but the underlying function they provide is the same
Insert table from here:

[MSTest does support data driven tests](https://msdn.microsoft.com/en-us/library/ms182527.aspx) but this isn't quite the same as nUnits TestCases attribute. Fortunately MSTest v2 provides a comparable option in the form of [DataTestMethod and DataRow attributes](https://blogs.msmvps.com/bsonnino/2017/03/18/parametrized-tests-with-ms-test/).

So quirks aside, MSTest worked really well inside of Visual Studio, everything was available in the Test Explorer, I could re-run all tests, run only new tests and debug difficult to setup test.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/01-test-explorer.png){: .align-center}

The coding exercise stipulated

> Build file: Please provide an automated build file that compiles your code and runs the tests. For java submissions, a Gradle or Maven build file is ideal. Submissions without an automated build will not be accepted.

### Calling from the command line
Calling MSTest from the command line should be easy, the [documentation is clear](https://msdn.microsoft.com/en-us/library/ms182489.aspx), it ought to be a case of running the program with the correct command line switches. On the face of it I should be able to open the Visual Studio command prompt, navigate to the root of my solution, build my solution and run MSTest from the command line

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/02-mstest_from_commandline.png){: .align-center}

Oh dear, epic fail, after burning up around three hours I switched tactics having read an article from an [MSDN page](https://msdn.microsoft.com/en-us/library/ms182486(v=vs.140).aspx) stating that VSTest.Console.exe can be used from the command line instead of MSTest.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/03-vstest-commandline
.png){: .align-center}

And with that I'm happy to use VSTest.Console.exe because ultimately this is just for a coding kata, on my development environment I'll stay within the confines of Visual Studio, if I was running my own build server then I'd invest more time.
