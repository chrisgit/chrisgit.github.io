---
title: "Chef: Attributes (part 2)"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef attributes are complicated, knowing when to set them is not easy, this article is not intended to replace the excellent documentation found at the OpsCode web site but rather to compliment it.
<!--more-->
## Using Attributes in a Recipe

An attribute can be used in a recipe without previously having been declared and assigned a value; when an undeclared attribute is used in a recipe it's value appears to be Empty

```ruby
# The attribute ['AttributeDoesNotExist'] has not been declared or assigned a value but using it does not cause the Chef run the error
log "ShowAttributeNotSet" do
  message "This attribute value for AttributeDoesNotExist is #{node['AttributeDoesNotExist']}"
  level :info
end
```

The attributes are associated with the node object and at runtime are stored as a Ruby Hashmap; this means it is possible to write Chef code to determine if an attribute exists in the same way that you would with any other Ruby HashMap

```ruby
# Various methods to Check to see if the attribute exists
node['AttributeDoesNotExist'].nil?
node.attribute?('AttributeDoesNotExist')

# When the attribute has not been declared or assigned checking if the attribute is empty will result in a Chef exception being thrown
node['AttributeDoesNotExist'].empty?
 
# Results in
NoMethodError
-------------
undefined method `empty?' for nil:NilClass
 
# Therefore if unsure whether the attribute exists check before using it
```

So far we've seen simple use of attributes; the structure that holds the attributes is a HashMap which is similar to a jagged array, this means that an element in the attributes collection can be a parent to another element in the attributes collection, knowing this we can nest attributes as below

```ruby
# Declare the attributes, am and pm attributes have the parent of Greeting
node.default['Greeting']['am'] = "Good morning"
node.default['Greeting']['pm'] = "Good afternoon"
 
# Use the nested attributes in a Cookbook
if Time.now.hour < 12 then
  useGreeting = node['Greeting']['am']
else
  useGreeting = node['Greeting']['pm']
end

log "LogGreeting" do
  message "#{useGreeting} Chef client is now running ..."
  level :info
end
```

When declaring and assigning a nested attribute the parent attribute does not need to exist; in the above example we created ['Greeting']['am'] without having declared ['Greeting'].

However if you try to use a nested attribute before it has been declared and assigned a value Chef will throw an exception

```ruby
# Greeting.am has NOT been declared and assigned using it in code like so ...
node['Greeting']['am']
 
# Will result in this error
 
NoMethodError
-------------
undefined method `[]' for nil:NilClass
```

Using Ruby we can test to see if the nested attribute exists before using it.

```ruby
# First check the parent attribute exists then check for the child attribute
message = ""
if node['AttributeDoesNotExist'].nil?
  message = "Parent attribute is not declared"
elsif node['AttributeDoesNotExist']['SubAttributeDoesNotExist'].nil?
  message = "Child attribute is not declared"
end
 
# Can also use node.attribute('key') or node.has_key?('key') or node.key?('key'), in fact any syntax relating to Ruby hash
```

If your attribute nesting is deep then defensive code can become convoluted
```ruby
# Checking a deeply nested attribute
message = ""
if node['TopLevelAttribute']
  if node['TopLevelAttribute']['SecondLevelAttribute']
    if node['TopLevelAttribute']['SecondLevelAttribute']['ThirdLevelAttribute']
      message = node['TopLevelAttribute']['SecondLevelAttribute']['ThirdLevelAttribute']
    end
  end
end
```

If nesting is deep you can wrap the code with a Ruby try ... catch, if the attribute does not exist an exception will be thrown which can be caught, an alternative is to use JSON paths to determine if the attribute exists

```ruby
# Access the attribute in Ruby try...catch, in this example we end the Chef session if accessing the attribute throws an exception
begin
  message = node['AttributeDoesNotExist']['SubAttributeDoesNotExist']
rescue 
  raise "Error: Failed to locate data bag for #{node['YourCompany']['machine_id']}."
end
 
# Using a JSON path to determine if the nested attribute exists
message = "Default"
if node['$.AttributeDoesNotExist.SubAttributeDoesNotExist'].nil?
  message = "Attribute is not declared"
end
```

Update: Chef Sugar from Seth Vargo has a [deep_fetch](https://github.com/sethvargo/chef-sugar#node) method which will check to see if a nested attribute exists; saving you from writing your own defensive code or embedding Ruby code in a try .. catch.

## When is an Attribute evaluated

From Chef version 11 onwards the way attributes were evaluated and assigned became much simpler, before Chef version 11 the attribute evaluation appeared to be in an unknown order.

To be able to understand when the attributes are evaluated it is best to know what is involved in a Chef run (the image and majority of the text describing the Chef run has been taken from the OpsCode documentation).

![01-chef-attributes]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/01-chef-attributes.jpg){: .align-center}

### Get configuration data
The chef-client gets process configuration data from the client.rb file on the node, and then gets node configuration data from Ohai. One important piece of configuration data is the name of the node, which is found in the node_name attribute in the client.rb file or is provided by Ohai. If Ohai provides the name of a node, it is typically the FQDN for the node, which is always unique within an organization.

### Authenticate to the Chef Server
The chef-client authenticates to the Chef server using an RSA private key and the Chef Server API. The name of the node is required as part of the authentication process to the Chef server. If this is the first chef-client run for a node, the chef-validator will be used to generate the RSA private key.

### Get, rebuild the node object
The chef-client pulls down the node object from the Chef server. If this is the first chef-client run for the node, there will not be a node object to pull down from the Chef server. After the node object is pulled down from the Chef server, the chef-client rebuilds the node object. If this is the first chef-client run for the node, the rebuilt node object will contain only the default run-list. For any subsequent chef-client run, the rebuilt node object will also contain the run-list from the previous chef-client run.

### Expand the run-list
The chef-client expands the run-list from the rebuilt node object, compiling a full and complete list of roles and recipes that will be applied to the node, placing the roles and recipes in the same exact order they will be applied. (The run-list is stored in each node object’s JSON file, grouped under run_list.)

### Synchronize cookbooks
The chef-client asks the Chef server for a list of all cookbook files (including recipes, templates, resources, providers, attributes, libraries, and definitions) that will be required to do every action identified in the run-list for the rebuilt node object. The Chef server provides to the chef-client a list of all of those files. The chef-client compares this list to the cookbook files cached on the node (from previous chef-client runs), and then downloads a copy of every file that has changed since the previous chef-client run, along with any new files.

### Reset node attributes
All attributes in the rebuilt node object are reset. All attributes from attribute files, environments, roles, and Ohai are loaded. Attributes that are defined in attribute files are first loaded according to cookbook order. For each cookbook, attributes in the default.rb file are loaded first, and then additional attribute files (if present) are loaded in lexical sort order. All attributes in the rebuilt node object are updated with the attribute data according to attribute precedence. When all of the attributes are updated, the rebuilt node object is complete.
Any dependant Cookbooks attribute files will be evaluated before the top level Cookbook file attributes are evaluated, this makes it easy to modify default cookbook values when writing wrapper cookbooks.

### Compile the resource collection
The chef-client identifies each resource in the node object and builds the resource collection. All libraries are loaded (to ensure that all language extensions and Ruby classes are available). And then all attributes are loaded. And then all lightweight resources are loaded. And then all definitions are loaded (to ensure that any pseudo-resources are available). Finally, all recipes are loaded in the order specified by the expanded run-list.
Any attributes assigned in a recipe with modify any values declared and assigned in the attribute files unless the attribute precedence is used.

### Converge the node
The chef-client configures the system based on the information that has been collected. Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence.

### Update the node object, process exception and report handlers
When all of the actions identified by resources in the resource collection have been done, and when the chef-client run finished successfully, the chef-client updates the node object on the Chef server with the node object that was built during this chef-client run. (This node object will be pulled down by the chef-client during the next chef-client run.) This makes the node object (and the data in the node object) available for search.
The chef-client always checks the resource collection for the presence of exception and report handlers. If any are present, each one is processed appropriately.

### Stop, wait for the next run
When everything is configured and the chef-client run is complete, the chef-client stops and waits until the next time it is asked to run.

## Lazy Attributes
There are two execution phases in Chef, the first execution phase evaluates and compiles your resources (compile the resource collection), the second execution phase is the converge (converge the node) which actions your resource.

When using attributes the attribute value is used in the first phase of execution to construct the resource

```ruby
# attributes/default.rb
default['YourCompany']['S3_bucker'] = "dev-apps"
default['YourCompany']['devtools']['ansicon'] = "ansi166.zip"
 
# recipes/default.rb
s3_file "c:\\devtools\\#{node['YourCompany']['devtools']['ansicon']}" do
	remote_path "/#{node['YourCompany']['devtools']['ansicon']}"
	bucket node['YourCompany']['dev_apps_bucket']
	aws_access_key_id aws['aws_access_key_id']
	aws_secret_access_key aws['aws_secret_access_key']
	action :create
	notifies :run, "execute[Install Ansicon]", :delayed
end
 
# After first phase of execution the resource above looks like this
s3_file "c:\\devtools\\ansi166.zip" do
	remote_path "/ansi166.zip"
	bucket dev-apps
	aws_access_key_id aws_key
	aws_secret_access_key aws_secret
	action :create
	notifies :run, "execute[Install Ansicon]", :delayed
end
```

Sometimes you might want to delay the evaluation of a attribute, perhaps a recipe in your run list executes AFTER your resource is compiled but will change the value of an attribute used by your resource, maybe the value of the attribute is changed at the time of the converge.In these cases you can use Lazy Attribute Evaluation.

```ruby
# The syntax for using lazy evaluation is attribute_name lazy { code_block }, an example of lazy is below
 
some_resource "#{some_resource_name}" do
  file_location lazy { node['drive']['path']['uuid'] }
  action [:do_stuff]
end
 
# The lazy keyword uses the Chef::DelayedEvaluator
```

## Node Attributes on the Chef server
The node has been mentioned throughout this documentation and as we can see from the Chef execution run the node attributes are sent back to the server at the end of the Chef run but what does this mean exactly?

The next few steps will demonstrate the node object on the Chef server by provisioning a Virtual Box Windows image with Vagrant and performing a Chef client run
Clear out any knowledge the the Chef server has of your VM

![02-chef-node-attributes]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/02-chef-node-attributes.jpg){: .align-center}

Change the run list in the Vagrant file so that there are no actions for Chef to perform, this will ensure the Chef client registers with the Chef server but does not run any Cookbooks (EMPTY RUN LIST)

![03-chef-empty-run-list]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/03-chef-empty-run-list.jpg){: .align-center}

The Chef server now has the client registered, the client chris-sullivan-vmWindows2012 has been added

![04-chef-empty-run-register-client]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/04-chef-empty-run-register-client.jpg){: .align-center}

and because we have completed a Chef run we also have the node on the Chef server 

![05-chef-empty-run-create-node]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/05-chef-empty-run-create-node.jpg){: .align-center}

but there are no cookbook specific attributes, just those attributes that gathered automatically by Ohai

![06-chef-empty-run-ohai-attributes]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/06-chef-empty-run-ohai-attributes.jpg){: .align-center}

Modify the run list to execute a base cookbook, lets say that it is called os_win2012, reprovision it with Vagrant and Chef

![07-chef-provision-with-run-list]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/07-chef-provision-with-run-list.jpg){: .align-center}

When the Chef run completes the node will be updated with attributes from the Cookbooks and recipes that have run.

![08-chef-cookbook-attributes]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/08-chef-cookbook-attributes.jpg){: .align-center}

Chef recipes that use local variables to store values calculated during the Chef client run will not be persisted away in the node; this is fine but those calculated values are unknown after the Chef run has completed, if you need to see a the result of a calculated value I recommended the values is stored on the Node. For secrets, passwords or dynamically generated code you should not store those in the Node (unless they are encrypted first).

If you don't have access to the Chef server GUI then the node attributes can be displayed using the Chef knife command (knife node show <node_name>)

![09-knife-to-view-node-attributes]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-10-05-chef-attributes/09-knife-to-view-node-attributes.jpg){: .align-center}

### Temporary attribute values
Sometimes attributes are used to store values generated at run time which are then used as parameters in resources.

At the end of the Chef run all of the attributes are sent back to the Chef server, the attributes sent back to the Chef server can be viewing when querying the node with knife or using the Chef UI, therefore do not store temporal values or secrets in attributes.

If you want to use an attribute to store a value for run time (that needs to persists across recipes or cookbooks) then prefer to use node.run_state or consider using attribute whitelisting or blacklisting to remove the attributes so they are not sent back to the Chef server.
