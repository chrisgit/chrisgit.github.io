---
title: "MSBuild: Building solution files"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - msbuild
  - C#
---

For the past couple of years I've been working in a DevOps role bridging any gap between development and operations to smooth out the continous delivery pipeline. The company I worked for has a mixture of Windows and Linux operating systems in production so during that time I wrote scripts in both bash and PowerShell, bespoke DSL's such as Terraform and Chef and was lucky enough to choose languages for various services; selecting from Ruby, Python or Go depending on what the deliverable was. Prior to my stint in operations I was a C# guy, having first learnt the language in 2000 working for Microsoft.

<!--more-->

In looking for my next move I was asked to carry out a code kata, the instructions specified that the solution should be zipped but include an automated build or more specifically

> Build file: Please provide an automated build file that compiles your code and runs the tests. For java submissions, a Gradle or Maven build file is ideal. Submissions without an automated build will not be accepted.

Well it's been a few years since I tackled C#, luckily the syntax came rolling back and the kata was done in a few hours but I was confused as to what to do with regards to automated builds. In my time working DevOps I'd implimented Jenkins to perform CI/CD later moving on to using JenkinsFiles, for Github and other projects I'd used a mixture of CircleCI, Travis and AppVeyor. As I had no idea what the target machine would look like I set about thinking of the most basic options.

## Build options
Back in the day my C# code had to be compiled using the C# command line compiler csc.exe, while the command line compiler is good it is not Enterprise grade so to reduce the complexity we moved on to using C# response files. The C# response file will usually have an *.rsp extension and contain all of the instructions required to build your application.

Since the invention of Visual Studio though, it's hard to make a case for using the C# command line compiler. After leaving Microsoft the company I worked for originally built their applications once a week by hand using Visual Studio; before moving on to overnight builds using TFS.

Fast foward a few years and I had joined another company with a different build process, they had MSBuild projects, all lovingly crafted by hand and a dedicated build server running CruiseControl.Net; the office had several radiators showing the build status, red for failure, amber for building and green for good. MSBuild is a great tool but I always thinking of it as a tool from a certain era; the everything XML era in fact, it is very powerful but not very easy to read or debug.

Fast forward again to my time in DevOps and my build script of choice for all language builds was generally Rake. Originally written for performing builds of Ruby applications Rake is a simple to read internal DSL, in order to run a Rake build you will need Ruby installed on your server.

Do back to the coding kata, I can either using csc, response files, MSBuild or opt for a Rake like script designed specifically for C#, something like Cake (https://cakebuild.net/). MSBuild was the obvious choice but I'd only every worked on solutions that were written by hand so was reluctant to venture into that terratory until I did a bit of searching, it turns out thatg MSBuild will take your Visual Studio solution file and build the project based on that.

## Using MSBuild to build a solution file
Simply open the Visual Studio developer command prompt, navigate to the root of your project and type MSBuild or MSBuild <name of solution>

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/01-msbuild-solution.png){: .align-center}

When using MSBuild with a solution file you can also specify a target and a build configuration, such as msbuild /t:Build /p:Configuration=Release

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/02-msuild-solution-release-config.png){: .align-center}

The solution file doesn't even contain a target, it isn't even XML so how can you specify a target at all? MSBuild actually generates an MSBuild file in memory and executes that; to see the generated file you will need to add an environment variable, set msbuildemitsolution=1. Now running MSBuild will generate a metaproj file

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/03-msbuild-dynamic-file.png){: .align-center}

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/04-metaproj.png){: .align-center}

The metaproj file resembles the XML MSBuild files that we all know and love so you can navigate to the different targets.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-28-msbuild-building-solution-files/05-build-target.png){: .align-center}

## MSBuild and Nuget restore
New for MSBuild 15.1 and above is the ability to restore Nuget packages. This feature was documented here: [https://docs.microsoft.com/en-us/nuget/consume-packages/package-restore](https://docs.microsoft.com/en-us/nuget/consume-packages/package-restore)

If you are building a C# solution with MSBuild from a solution file you can specify the restore target with /t:restore.

### Warning for MSTest projects
So, it's not all good. If you have a project that uses MSTest and use the MSBuild restore target it will not download the appropriate Nuget packages for you.
The question was raised here: [https://developercommunity.visualstudio.com/content/problem/32155/msbuild-trestore-doesnt-work-with-mstest-projects.html](https://developercommunity.visualstudio.com/content/problem/32155/msbuild-trestore-doesnt-work-with-mstest-projects.html) with the answer that Nuget packages will not be restored for MSTest projects and this is by design.
