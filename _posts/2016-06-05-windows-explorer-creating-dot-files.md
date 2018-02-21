---
title: "Windows Explorer: Creating dot files"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - Windows
  - Explorer
  - Command
---

Dot files, the files in your folder that begining with a dot, files like .gitignore, you create those in the command prompt don't you?

<!--more--> 

Well, something I learnt today, you can create a file (or folder) begining with a dot in Windows Explorer.

### Dot Files and Folders from Windows Explorer
Right click in Windows Explorer, select New -> choose your file type, Windows Explorer will then allow you to rename the file, typing ".gitignore" will not work, you will likely get

![01-windows-file-rename-error]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-06-05-windows-explorer-creating-dot-files/01-windows-file-rename-error.jpg){: .align-center}

To save your dot file add an extra dot at the end of the filename, therefore instead of typing ".gitignore" type ".gitignore."

The same thing works for folders, right click in Windows Explorer, select New -> Folder and add an extra dot to the end of your folder name.

![02-windows-folder-dot-name]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-06-05-windows-explorer-creating-dot-files/02-windows-folder-dot-name.jpg){: .align-center}
