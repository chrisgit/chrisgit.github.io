---
title: "Git: viewing previous versions of files"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - git
---

Git log allows you to see the history of the file, occasionally you want to see the state of the file at a particular check in so you might checkout your code at a particular hash but it is a lot of work if you only want to see a individual file. 
<!--more-->

## Showing the complete file
### Using git show
[git show](https://git-scm.com/docs/git-show) is run from the command line and allows you to show various types of objects, we can use it to view our file at a particular check in.

Obtain the hash of the check in with git log, the file I am interested in is lib/bitmap.rb
```
git log lib/bitmap.rb
```

![01-git-log-bitmap]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/01-git-log-bitmap.jpg){: .align-center}

Use git show to display the file at the time of the check in with git show <hash>:<file>
```
git show 5813413517c1a9fb5dce56a77dc1c58027d8bb71:lib/bitmap.rb
```

![02-git-show-bitmap-commit-hash]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/02-git-show-bitmap-commit-hash.jpg){: .align-center}

We can also use a relative check in to HEAD by using a tilda an number. To see the file four check ins before HEAD you would type
```
git show HEAD~4:lib/bitmap.rb
```

![03-git-show-bitmap-head-minus-four]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/03-git-show-bitmap-head-minus-four.jpg){: .align-center}

### Gitk
[Gitk](https://git-scm.com/docs/gitk) is a GUI tool for Git.

On Windows gitk is installed as part of the Git bash tools, on Ubuntu if you have Git installed but not Gitk you can install it with the apt package manager

```
sudo apt-get install gitk
```

As an alternative to Git bash you could use the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

To view a single file with gitk type gitk <file>
```
gitk lib/bitmap.rb
```

The gitk GUI opens up and displays the commit log; choose the commit you want, Gitk will display the difference or the entire file, to switch between viewing the diff or entire file use the patch or tree radio buttons. Patch will show differences, if you select a file in tree the entire file is displayed.

![04-gitk-patch-or-tree]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/04-gitk-patch-or-tree.jpg){: .align-center}

## Visual SourceTree
[Atlassian SourceTree](https://www.sourcetreeapp.com/) is a really nice (and free!) git GUI tool.

To display a file at a particular version from the menu select View -> File Status View

![05-sourcetree-file-status-view]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/05-sourcetree-file-status-view.jpg){: .align-center}

change the drop down from Pending -> All

![06-sourcetree-file-pending-all]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/06-sourcetree-file-pending-all.jpg){: .align-center}

search for your file or select your file (in unstaged), right-click and choose "Log Selected"

![07-sourcetree-log-selected]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/07-sourcetree-log-selected.jpg){: .align-center}

Select your commit, the diff will automatically be displayed, to view the entire file right click the commit and choose "Open selected version"

![08-sourcetree-log-open-selected]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-14-git-viewing-previous-versions/08-sourcetree-log-open-selected.jpg){: .align-center}
