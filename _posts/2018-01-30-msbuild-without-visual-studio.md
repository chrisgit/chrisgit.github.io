---
title: "MSBuild: Building solutions without Visual Studio installed"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - msbuild
  - dotnet
---

Sometimes it's easy to forget just what Visual Studio actually does for you, as it turns out quite a lot.

This topic might seem easy if you run your own build server but for those that don't how do you start with a vanilla operating system install and get to a position where you can build a .Net project without first installing Visual Studio?

<!--more-->

## Build Tools
Download build tools from the [Visual Studio download page](https://www.visualstudio.com/downloads/)

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-30-msbuild-building-solution-files/01-download-build-tools.png){: .align-center}

Install the tools and select Web development build tools, this option also selects the .Net frameworks for installation, should you wish you can customise what is instaleld by selecting Individual Components or Language Packs tabs.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-30-msbuild-building-solution-files/02-install-build-tools.png){: .align-center}
![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-30-msbuild-building-solution-files/03-install-completed.png){: .align-center}

After installing the build tools you will get a new command prompt in the start menu, called “Developer Command Prompt for VS 2017”. 

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-30-msbuild-building-solution-files/04-developer-command-prompt.png){: .align-center}

This command prompt runs a script to create all of the environment variables required to run the build tools.

Go to your folder with your solution sln file, and type MSBuild to build your solution.

## Nuget packages
If you use Nuget packages these will need to be restored, fortunately Nuget supply you with a single EXE to run and it can be downloaded from here: [https://www.nuget.org/downloads](https://www.nuget.org/downloads).

Save the Nuget.exe file to a folder in your path, or the same location that MSBuild is stored, if you downloaded the Visual Studio 2017 build tools then it is likely to be stored in %programfiles(x86)%\Microsoft Visual Studio\2017\BuildTools\MSBuild\15.0\Bin folder.

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2018-01-30-msbuild-building-solution-files/05-msbuild_location.png){: .align-center}

You'll see from above that I have two versions of MSBuild installed, it is probaby because the locations and names of certain files have changed for Visual Studio 2017 so you will have to check your exact installation location. 

Before running MSBuild download the nuget packages by typing “nuget restore”; the “packages” folder will be created and contain the specified packages.

## Command Window
If you opened the command window instead of the Developer Command Window you can run the Visual Studio batch file to setup the appropriate environment by typing

```
"C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\Common7\Tools\vsdevcmd.bat"
```

The easiest way to type that is to enter "C:\P and use the tab key to cycle through each item in the path!

Once you've run the batch file all of the environment variables are set and you can run MSBuild.
