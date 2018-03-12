---
title: "Chef: the Ruby bits"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef is a DSL built on top of Ruby; it is classified as an [internal DSL](https://martinfowler.com/bliki/InternalDslStyle.html); this means that you can write Ruby code and take advantage of all of the great tooling that comes with this language.

Here are a few guidelines when using pure Ruby code with Chef
<!--more-->
## Syntax Checks
- Run a simple syntax check by running ruby -c my_cookbook_file.rb
- Run [Rubocop](https://github.com/bbatsov/rubocop) against pure Ruby code
- (Optional) Run [RubyCritic](https://github.com/whitesmith/rubycritic) or [Flog](http://ruby.sadi.st/Flog.html) or [Flay](http://ruby.sadi.st/Flay.html)

## General
- Try and keep cookbook recipes clear of Ruby code, place Ruby code in libraries and use RSpec to test any classes or methods.
- If your method returns true or false then convention is to end the method name with a question mark, i.e. Dir.exist?
- If your method modifies the underlying data structure contained in a class then use the exclaimation mark, i.e. s = 'hello'; s.upcase!

## Variables
- Do not use global variables (which are signified by a dollar sign as the beginning of the variable name).
- If you use any built in Ruby variables do NOT use the abbreviated short names, i.e. do not use $: use $LOAD_PATH
- Use the English gem if necessary to convert abbreviated names to descriptive names, 

## Built In Classes
Some Ruby classes have singular and plural methods, prefer singular methods (as it reads better) so some of the plural methods are marked for deprecation;
e.g. use File.exist? over File.exists?  

NB: Do not get confused with the Ruby on Rails pluralize method! The above remark refers to method names. The pluralize method can be found by require 'active_support/inflector'

A good resource for [deprecated methods](http://batsov.com/articles/2014/02/05/a-list-of-deprecated-stuff-in-ruby/) can be found from the author of Rubocop
