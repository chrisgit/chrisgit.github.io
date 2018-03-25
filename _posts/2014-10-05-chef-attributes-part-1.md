---
title: "Chef: Attributes (part 1)"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef attributes are complicated, knowing when to set them is not easy, this article is not intended to replace the excellent documentation found at the OpsCode web site but rather to compliment it.
<!--more-->
## Chef Attributes
An attribute is basically a setting.

The attribute is accessed via a key and contains a value, the attributes resemble a jagged array or hash, examples of attributes are as follows

```ruby
default['Greeting'] = "Good day"
default['Greeting']['am'] = "Good morning"
default['Greeting']['pm'] = "Good afternoon"
```

An attribute can be declared and assigned in a cookbook (or recipe), environment or role.

Attributes declared in a cookbook or recipe are stored in a node object on the Chef server (in Chef terms a node is defined as any physical, virtual, or cloud machine that is configured to be maintained by a chef-client).

When the Chef client executes the the node object attributes are represented as a Ruby HashMap.

## Setting Attributes
In most cases an attribute will be declared and assigned in a Cookbook rather than a Role or Environment.

When declaring an attribute in a recipe it is usually placed in an attribute file (found in the Cookbooks attributes folder).

```ruby
# Declaring and assigning an attribute in an attribute file, below declares an attribute called Greeting, node maybe omitted as it is implied  
node.default['Greeting'] = "Good day"
 
# The attribute is associated with the node object; when declared and assigned in an attributes file the node keyword is implied, syntax below is valid
default['Greeting'] = "Good day"
 
# An attribute can be declared and assigned in a recipe file but requires the node keyword as follows
node.default['Greeting'] = "Good day"
```

The use of the word default is setting an attribute precedent level, generally a default attribute has the lowest value in order of precedence and means the value can be modified in a Recipe, Role or Environment.

An attribute in attributes/default.rb file will be evaluated when the Cookbook is included in a Chef run.

An attribute in a specific attribute file is only evaluated when the corresponding recipe is included, for example the attributes/PowerShell.rb will only be evaluated when the recipe/PowerShell.rb is included in a Chef run.

Setting attributes for Environment or Roles is done by editing the Role or Environment directly on the Chef server, the preferred way of changing Role or Environment attributes is to use a file and upload it to the Chef server. If using an Environment or Role file then the attributes must be entered in JSON syntax.

Environment and Role attributes can be added as default_attributes or override_attributes, this related to the attribute order of precedence

```ruby
# Here we are setting the default_attributes of a role in a Role file
default_attributes(
    :Greeting => {
	:am => "Good morning",
	:pm => "Good afternoon" }
)
 
# Updating the Chef server with the role attributes when using an Role file (recommended)
knife role from file <file_name>

# Updating the Chef server with the Environment attributes when using an Environment file (recommended) 
knife environment from file <file_name>
```

## Supported Datatypes

An attribute value can be any data type supported by [Ruby Hash](http://ruby-for-beginners.rubymonstas.org/built_in_classes/hashes.html).

## Automatic Attributes

Some attributes are automatically gathered by Chef; the automatic attributes are environment values extracted from the node such as ipaddress or domain. The automatic attributes are gathered by a Chef utility program installed on the node called Ohai.

The attributes and values collected by Ohai cannot be modified.

NB: The name of a node is required as part of the authentication process to the Chef server. The name of a node can be obtained from the node_name attribute in the client.rb file or by allowing Ohai to collect this data during the chef-client run. When Ohai collects this data during the chef-client run, it uses the FQDN name of the node (which is always unique within an organization) as the name of the node.Using the FQDN as the node name, and then allowing Ohai to collect this information during each chef-client run, is the recommended approach and the easiest way to ensure that the names of all nodes across the organization are unique. 
