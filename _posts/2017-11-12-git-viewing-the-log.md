---
title: "Git: viewing the log"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - msbuild
  - dotnet
---

There are very few Git commands that I use on a daily basis, typically the commands I'm most familiar with are
- retrieve latest source files
- add files
- commit files
<!--more-->
- create branch
- switch branches
- merge branch
- push changes
- remove branch

Most of my Git experience has been in Windows using [SourceTree from Atlassian](https://www.sourcetreeapp.com/) or the Git integration in [Visual Studio Code](https://code.visualstudio.com/docs/introvideos/versioncontrol) 

![01-download-build-tools]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-12-git-viewing-the-log/01-git-in_visual-studio-code.jpg){: .align-center}

As part of a recent investigation I installed Git bash and started to use the command line a little more.

If you know Git you'll know there is more than one way to perform the same function, here are some ways to query the Git log!

## Git log
Just want to view the check in comments
```
git log -- file
```

Check in comments and changes
```
git log --follow -p -- file
```

or

```
git whatchanged -p file
```

If a colon appears Git is just paginating using more, just press return for a new page or q to quit.

## Gitk

On Windows gitk is installed as part of the Git bash tools, on Ubuntu if you have Git installed but not Gitk you can install it with the apt package manager

```
sudo apt-get install gitk
```

As an alternative to Git bash you could use the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

View history of a single file
```
gitk file
```

## Visual SourceTree

SourceTree has a nice visual view of branches and check ins from the menu select View -> Log View 

![02-sourcetree-visual]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-12-git-viewing-the-log/02-sourcetree-visual.jpg){: .align-center}

This view can be a bit overwhelming if you just want to look for specific files or changes go to File Status and select working Copy; ensure that you have all files selected.

![03-sourcetree-select-files]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-12-git-viewing-the-log/03-sourcetree-select-files.jpg){: .align-center}

In your unstaged files select the file you want, from the menu select Action -> Log Selected or right click on the file and choose Log Selected

![04-sourcetree-log-view]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-12-git-viewing-the-log/04-sourcetree-log-view.jpg){: .align-center}
 
Just one warning, SourceTree can slow down for large projects or those with a lot of files, other than that, it's great!

### Git log cheat sheet (grab details from Jira and Git document)

#### Basic log view
```
git log
git log --oneline –decorate
```

#### Range of days
```
git log --since "NOV 20 2017" --until "NOV 22 2017" --pretty=format:"%h %an %ad %s"
git log origin/Branch --since "NOV 20 2017" --until "NOV 22 2017" --graph --oneline --decorate
```

#### Single day
```
git log origin/Branch --after "NOV 22 2017 00:00" --before "NOV 22 2017 23:59" --graph --oneline –decorate 
```

#### Compare two branches
```
git log Branch1...Branch2 --oneline	
git log Branch1...Branch2 –oneline
```

#### Commits with X3 and JIRA in the description that are 5 days or less
```
git log origin/Branch --grep="X3-" -grep="JIRA" --since=5.day 
git log origin/Branch --grep="X3-" -grep="JIRA" -i --since=5.day (-i to ignore case)
```

#### Grep to extract information
```
git log --oneline | grep -o "Merge conflict resolution" (only Merge Conflict resolution)
git log --oneline | grep -v "Merge conflict resolution" (-v not Merge conflict resolution)
```

#### View branches visually
```
git log --graph --pretty=oneline --abbrev-commit
```

## More resources
Unsurprisingly the best place for learning Git is from [Github](https://github.com/). GitHub is a web-based hosting service for version control using git.

Some [useful links]((https://git-scm.com/doc)) in Github
- Git manual
- Git Pro book (v1 available to download, v2 free to read online)

Or how about having a little [try out of git](https://try.github.io/levels/1/challenges/1)

Like me, if you have used Atlassian's SourceTree or [BitBucket](https://bitbucket.org/product) you will realise that they too offer [some great articles](https://www.atlassian.com/git).

Also great resources elsewhere like this Visual playpen [https://learngitbranching.js.org/](https://learngitbranching.js.org/) or a generally good sandboxed learning environment such as [katacoda](https://www.katacoda.com/courses/git)

Git is a great source control system which is easy to get started with but has plenty of depth that you never stop learning it.
