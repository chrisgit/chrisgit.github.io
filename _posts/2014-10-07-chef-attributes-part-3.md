---
title: "Chef: Attributes (part 3)"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef attributes are complicated, knowing when to set them is not easy, this article is not intended to replace the excellent documentation found at the OpsCode web site but rather to compliment it.
<!--more-->

## Attribute precedence

An attribute can be declared and assigned in an attributes file (found in the Cookbooks attributes folder), a recipe file, directly on the node object, as part of the Chef role, as part of the Chef environment or collected automatically by Chef client (using Ohai).

If an attribute is declared and assigned a value in an attribute file, a recipe file and a Role which value is actually used?

```ruby
# In the attributes/default.rb
default['Greeting'] = "Hello"
 
# In the recipes/default.rb
node.default['Greeting'] = "Bonjour"
 
# In the role we have a default_attribute of ['Greeting'] = "Guten Tag!"
 
# When Chef runs what value is used? It's actually Guten Tag!
# The attributes/default.rb is processed first, at this point of the Chef client run the value of 'Greeting' is now "Hello"
# The recipe/default.rb is then processed next and changes the value of the attribute, at this point of the Chef client run the value of 'Greeting' is now "Bonjour"
# However, the role attribute is higher in the attribute order of precedence and wins! 
```

The example above uses the keyword default when declaring and assigning an attribute but there are other keywords that can be used when an attribute is declared and assigned.

The keywords that we have for declaring attributes are default, force_default, normal, override and force_override, when writing a Cookbook all of the keywords can be used in place of the word default.

```ruby
# Using attribute precedence is easy, simply specify the keyword default, force_default, normal, override or force_override
# Setting an attribute with override in an attributes file
override['Greeting'] = "Bonjour"
 
# When setting an attribute in an attribute file 
node.override['Greeting'] = "Hello"
 
# Using force_override in a recipe
node.force_override['Greeting'] = "Hola" 
```

The keywords default, force_default, normal, override and force_override specify the order of precedence of the attribute, working out who wins is a bit like paper, scissors, stone but OpsCode have a table to describe the order of attribute precedence and attributes are always applied by the chef-client in the following order:

1.	A default attribute located in a cookbook attribute file
2.	A default attribute located in a recipe
3.	A default attribute located in an environment
4.	A default attribute located in role
5.	A force_default attribute located in a cookbook attribute file
6.	A force_default attribute located in a recipe
7.	A normal attribute located in a cookbook attribute file
8.	A normal attribute located in a recipe
9.	An override attribute located in a cookbook attribute file
10.	An override attribute located in a recipe
11.	An override attribute located in a role
12.	An override attribute located in an environment
13.	A force_override attribute located in a cookbook attribute file
14.	A force_override attribute located in a recipe
15.	An automatic attribute identified by Ohai at the start of the chef-client run

Care needs to be taken when using an attribute keyword other than default, below are some examples using the order of precedence table

```ruby
###############################
# Example 1.
###############################
# attributes/default.rb
override['Greeting'] = "Hello"
 
# recipe/default.rb
node.default['Greeting'] = "Bonjour"
 
# Winner is ... attributes/default.rb value as override keyword in Cookbook attribute file is higher than default

############################### 
# Example 2.
###############################
# attributes/default.rb
default['Greeting'] = "Hello"
 
# recipe/default.rb
node.default['Greeting'] = "Bonjour"

# Winner is ... recipe/default.rb value as Recipe runs AFTER attribute evaluation and both assignments use the same level of precedence


###############################
# Example 3.
###############################
# attributes/default.rb
default['Greeting'] = "Hello"
 
# recipe/default.rb
node.override['Greeting'] = "Bonjour"

# In the role we have a default_attribute of ['Greeting'] = "Guten Tag!"


# Winner is ... recipe/default.rb value as the Recipe uses the override setting, if the Role value is required then the Role needs to be set with override_attributes
 
#####################################################################################################
# Therefore we need to be careful when using the force_default, override and force_override settings.
#####################################################################################################
```

The attribute precedence is shown below in pictoral format

![10-chef-attribute-precedence-01]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/10-chef-attribute-precedence-01.jpg){: .align-center}

![10-chef-attribute-precedence-02]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/10-chef-attribute-precedence-02.jpg){: .align-center}

## Wrapper Cookbooks
If we want to use the community redisio cookbook and modify some of the default attribute values contained in the redisio cookbook we can call the cookbook directly and use our Cookbooks attribute file to modify the default attribute values or create a wrapper cookbook.

Should we chose to use a wrapper cookbook to modify attributes (or behaviour) it is likely to contain
```
# Create the wrapper cookbook from the command line using knife
knife cookbook create wrapped_redisio
 
# In the wrapped_redisio/attributes/default.rb
default['redisio']['mirror'] = "http://chris.server/"
default['redisio']['base_name'] = 'latest-redis-'
default['redisio']['artifact_type'] = 'redis.tar.gz'
 
# In the wrapped_redisio/recipes/default.rb
include_recipe 'redisio'
```

Now we can run recipe[wrapped_redisio] and the default settings in our wrapper cookbook will be used in place of those in the community redisio Cookbook.
Note: it is not necessary to use normal or override keywords with the attribute because dependent Cookbook attribute files are loaded and evaluated by the Chef Client before those of the calling Cookbook.

So why would I use a wrapper cookbook rather than just modify the attributes directly in my Cookbook? Quite simply for re-use and better visibility, if we see we are using the wrapped_redisio cookbook (in theory) should be easier to determine where the attribute values come from; as usual take each individual case separately and use common sense.

Due to the attribute precedence the recommendation when changing the value of an attribute in a dependant cookbook is to use default rather than override.

Where did my attribute value come from? Given the attribute order of precedence and all of the locations that an attribute can be set there will obviously be some head scratching to work out where at runtime the attribute value has come from. Fortunately OpsCode have provided a nice little method extension to the node object that accepts an attribute(s) as a parameter and returns an array representing the order of precedence.

```
# attributes/default.rb
default['Greeting'] = "Good day"
 
# recipes/default.rb
node.override['Greeting'] = "Hello"
 
# recipes/showgreeting.rb
greetingSetting = node.debug_value(:Greeting)
log 'greetingDebugInfo' do
	message greetingSetting.to_s
	level :info
end
 
# output, the order of precedence is clearly seen, in this case when we refer to the Greeting attribute the value "Hello" will be used
[2014-09-10T17:16:46+01:00] INFO: 
[["set_unless_enabled?", false], 
["default", "Good day"], 
["env_default", :not_present], 
["role_default", :not_present], 
["force_default", :not_present], 
["normal", :not_present], 
["override", "Hello"], 
["role_override", :not_present], 
["env_override", :not_present], 
["force_override", :not_present], 
["automatic", :not_present]]
 
# The above output is telling us that a default value has been set and that an override has been specified, when the code runs the value in the override is used as it has the higher precedence
# set_unless is used to only set a value if no other value existed, irrespective of the order of precedence of the existing value
```

What the debug output will not tell you is where the attribute value was set (i.e. the actual file in a recipe), therefore if you are including three recipes in your Cookbook updating the same attribute you will need to examine the logic and recipe run list order to know which recipe has set the attribute value.

## Object Inspection
We've seen when a Chef attribute can be used even if it hasn't been declared and assigned

```
# The attribute ['AttributeDoesNotExist'] has not been declared or assigned a value but using it does not cause the Chef run the error
log "ShowAttributeNotSet" do
  message "This attribute value for AttributeDoesNotExist is #{node['AttributeDoesNotExist']}"
  level :info
end
 
# Output is as below
[2014-09-11T17:23:18+01:00] INFO: This attribute value for AttributeDoesNotExist is
```

We've seen how the Chef client run throws an exception when an attribute hasn't been declared and assigned
```
# The attribute ['AttributeDoesNotExist']['SubAttributeDoesNotExist'] has not been declared or assigned a value but using it causes a Chef runtime error
log "ShowAttributeNotSet" do
  message "This attribute value for SubAttributeDoesNotExist is #{node['AttributeDoesNotExist']['SubAttributeDoesNotExist']}"
  level :info
end
 
# Chef run fails with
NoMethodError
-------------
undefined method `[]' for nil:NilClass
```

The nil:NilClass issue is caused because the top level attribute 'AttributeDoesNotExist' has NOT not been declared. The actual structure of attributes in Ruby is a hash containing another hash; Chef automatically ensures that each declared level of attributes is a hash, so the following is valid
```
node.default['AttributeDoesNotExist']['SubAttributeDoesNotExist'] = "Some Value"
```

Using object inspect we can see that even though we did not declare AttributeDoesNotExist is it present (and the value is a hash)
```
# So after a declaration and assignment of a nested attribute lets inspect the parent attribute with Ruby's object inspect
node.default['AttributeDoesNotExist']['SubAttributeDoesNotExist'] = "Some Value"
nestedAttribute = node['AttributeDoesNotExist'].inspect
Chef::Log.info("Object inspection of ['AttributeDoesNotExist'] #{nestedAttribute}")
 
# The output is as below, the top level attribute has been created and contains a single attribute ...
[2014-09-11T17:51:05+01:00] INFO: Object inspection of ['AttributeDoesNotExist'] {"SubAttributeDoesNotExist"=>"Some Value"}
```

In pure Ruby this is not possible
```
attribute = {}
attribute['One']['Two'] = 'Hello'
NoMethodError: undefined method `[]=' for nil:NilClass
        from (irb):3
        from C:/opscode/chefdk/embedded/bin/irb.cmd:19:in `<main>'
```

The hash key 'One' needs to be declared first.
```
attribute = {}
attribute['One'] = {}
attribute['One']['Two'] = 'Hello'
=> "Hello"
```

While not critical to know Ruby data structures it is helpful to know and be aware of them when writing Chef Cookbooks.

## Attribute Validation
Two popular features that are not part of Chef-Client are 
- attribute validation
- attribute auditing
However [Jamie Winsor (reset)](https://github.com/reset) has written an [attribute validator cookbook](https://github.com/reset/chef-validation).

Using the chef-validation cookbook is very simple; attributes can be validated at compile time or converge time; default is converge time and this is what I recommend as attribute values may change in recipes (by chef search) as part of the cookbook compile phase.

First thing to do is apply your attribute rules to the [metadata.rb](https://docs.chef.io/config_rb_metadata.html) file; below is an example of some attribute rules in the metadata.rb file, we are also including a dependency on Jamie's validation cookbook.
```
# metadata.rb
# Cookbook details
name             'mywindows'
maintainer       'Chris Sullivan'
maintainer_email 'n/a'
license          'All rights reserved'
description      'Installs/Configures myfirst'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.1.0'


# Dependencies
depends 'validation'
 
# Adds a title and description to a group of attributes within a namespace.
# Adds rules for mywindows cookbook BUT this is only names of attributes, they are not limited to being named mywindows; hence NVM.Python.Version
grouping "mywindows",
  title: "My First Chef Windows Application"
attribute "mywindows/log_level",
  required: "required",
  default: "debug",
  choices: [
    "debug",
    "fatal",
    "warn",
    "info"
  ]
attribute "mywindows/nvm/cookie",
  required: "required",
  default: "",
  type: "string"
attribute "YourCompany/Python/Version",
  required: "required",
  default: "",
  type: "string"

```

In your cookbook add some attributes, for this example we will not declare mywindows.yourcompany.cookie
```
# default.rb - Default attributes file ...
 
default['mywindows']['log_level'] = 'debug'
# Not sure we need mywindows.yourcompany.cookie let's comment it out
#default['mywindows']['nvm']['cookie'] = '*828212AFED8227ADCCB7'
default['mywindows']['install_path'] = 'C:\Temp'
```

In one of your cookbook recipes use the validate_attributes resource from the validation cookbook.
```
# Author:: A Developer
# Cookbook Name:: mywindows
# Recipe:: default
 
# Validate those attributes ....
validate_attributes "mywindows"
```

![11-chef-attribute-validation-fail]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/11-chef-attribute-validation-fail.jpg){: .align-center}

Doh! We commented out a mandatory attribute (ok, we KNEW that already), so let's add it back in to our default attributes file
```
# default.rb - Default attributes file ...

default['mywindows']['log_level'] = 'debug'
# We need mywindows.nvm.cookie ... it's mandatory
default['mywindows']['nvm']['cookie'] = '*828212AFED8227ADCCB7'
default['mywindows']['install_path'] = 'C:\Temp'
```

![12-chef-attribute-validation-pass]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/11-chef-attribute-validation-pass.jpg){: .align-center}
