---
title: "MSBuild: Building solution files"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - msbuild
  - C#
---

For the past couple of years I've been working in a DevOps role bridging any gap between operations and development, building Windows and Linux infrastruture, writing scripts and smoothing out the continous delivery pipeline to enable us to deploy software multiple times a day. Now I'm coming back to Microsoft and C#, having first learnt the language in 2000 working for Microsoft.
<!--more-->
To improve my skill level I undertook a coding kata with instructions specifying that the solution should include an automated build or more specifically

> Build file: Please provide an automated build file that compiles your code and runs the tests. For java submissions, a Gradle or Maven build file is ideal. Submissions without an automated build will not be accepted.

Well it's been a few years since I wrote anything significant in C#, luckily the syntax came rolling back and the kata was done in a few hours but I was confused as to what to do with regards to automated builds. In my time working DevOps I'd implimented Jenkins to perform CI/CD later moving on to using JenkinsFiles, for Github and other projects I'd used a mixture of CircleCI, Travis and AppVeyor. As I had no idea what a target machine would look like I set about thinking of the most basic options.

## Build options
Back in the day my C# code had to be compiled using the C# command line compiler csc.exe, while the command line compiler is good it is not Enterprise grade so to reduce the complexity we moved on to using C# response files. The C# response file will usually have an *.rsp extension and contain all of the instructions required to build your application.

Since the invention of Visual Studio though, it's hard to make a case for using the C# command line compiler. After leaving Microsoft the company I worked for originally built their applications by hand using Visual Studio once a week; later moving on to performing overnight builds using TFS.

Fast foward a few years and the build process has vastly improved using MSBuild projects, all lovingly crafted by hand and a dedicated build server running CruiseControl.Net; the office has several radiators showing the build status, red for failure, amber for building and green for good. MSBuild is a great tool but I regard it as a tool from a certain era; the everything XML era in fact. MSBuild scipts are very powerful but not very easy to read, ehance or debug.

Fast forward again my build script of choice for practically all language builds is generally Rake. Originally written for performing builds of Ruby applications Rake is a simple to read internal DSL, in order to run a Rake build you will need Ruby installed on your server.

Thoughts for completing the coding kata, I can either using csc, response files, MSBuild or opt for a Rake like script designed specifically for C#, something like Cake (https://cakebuild.net/). If you want to go native, MSBuild is still the obvious choice but let's face it, we don't want to build the XML file by hand now do we?

## Using MSBuild to build a solution file
It turns out thatg MSBuild can build your Visual Studio solution file.

Simply open the Visual Studio developer command prompt, navigate to the root of your project and type MSBuild or MSBuild <name of solution>

![01-msbuild-solution]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/01-msbuild-solution.png){: .align-center}

When using MSBuild with a solution file you can also specify a target and a build configuration, such as msbuild /t:Build /p:Configuration=Release

![02-msuild-solution-release-config]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/02-msuild-solution-release-config.png){: .align-center}

The solution file doesn't even contain a target, it isn't even XML so how can you specify a target at all? MSBuild actually generates an MSBuild file in memory and executes that; to see the generated file you will need to add an environment variable, set msbuildemitsolution=1. Now running MSBuild will generate a metaproj file

![03-msbuild-dynamic-file]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/03-msbuild-dynamic-file.png){: .align-center}

![04-metaproj]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/04-metaproj.png){: .align-center}

The metaproj file resembles the XML MSBuild files that we all know and love so you can navigate to the different targets.

![05-build-target]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/05-build-target.png){: .align-center}

## MSBuild and Nuget restore
New for MSBuild 15.1 and above is the ability to restore Nuget packages. This feature was documented here: [https://docs.microsoft.com/en-us/nuget/consume-packages/package-restore](https://docs.microsoft.com/en-us/nuget/consume-packages/package-restore)

If you are building a C# solution with MSBuild from a solution file you can specify the restore target with /t:restore.

### Warning for MSTest projects
So, it's not all good. If you have a project that uses MSTest and use the MSBuild restore target it will not download the appropriate Nuget packages for you.
The question was raised here: [https://developercommunity.visualstudio.com/content/problem/32155/msbuild-trestore-doesnt-work-with-mstest-projects.html](https://developercommunity.visualstudio.com/content/problem/32155/msbuild-trestore-doesnt-work-with-mstest-projects.html) with the answer that Nuget packages will not be restored for MSTest projects and this is by design.
