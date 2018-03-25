---
title: "Chef: Attributes (guidelines)"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef attributes are complicated, knowing when to set them is not easy, this article is not intended to replace the excellent documentation found at the OpsCode web site but rather to compliment it.
<!--more-->

## Some common sense guidelines

- Declare attributes in an attributes file
- Always create a default value for an attribute
- When declaring attributes inside the attributes file use the abbreviated default not node.default
- Prefer lower case attribute names
- Prefer snake case rather than camel case

### Declaration
Always declare a default value in one the cookbooks attributes files.

However there is a train of thought that if the attribute is a place holder that will be overridden by an environment (or role) attribute value or populated by Chef search it is fine to leave the attribute undeclared; the reason being the Chef client will error and abort with nil class [] error if the configuration is incorrect, regardless declaring attributes is strongly recommended.

#### Namespace
Use namespaces to separate attribute names across multiple cookbooks to avoid name clashes, for example if you require an attribute named 'InstallFolder' it might be an attribute name that is used across multiple cookbooks or community cookbooks.

Generally your cookbook attributes ought to start with company name followed by the feature; use the Cookbook name as the namespace, this will make the top level parent attribute easy to trace, for example attribute for your redis cookbooks could be named `<YourCompany>.<redis>`.

#### Use bracket format
Use Bracket format not dot notation
```
# Prefer
default['YourCompany']['spacewalk'] = true
 
# Over
default.YourCompany.spacewalk = true
```

#### Strings over Symbols
Prefer strings as attribute names not symbols
```
# Prefer 
default['YourCompany']['spacewalk'] = true
 
# Over
default[:YourCompany][:spacewalk] = true
```

#### Use single-quoted strings when interpolation not required
Use single quote strings unless interpolation is required
```
# Prefer
default['YourCompany']['server_name'] = 'spacewalk_proxy'

# Over 
default['YourCompany']['server_name'] = "spacewalk_proxy"
```

### Precedence
Use the default keyword when declaring and assigning attributes in attribute files, it makes it easier for a Recipe, Role or Environment to modify the value, there are of course exceptions to this; if your value really should NOT be modified under any circumstances.
```
# Prefer
default['YourCompany']['spacewalk'] = true
 
# Over
normal['YourCompany']['spacewalk'] = true
```

However, if you have specified a high precedent level it can normally be overridden with force_override (or settings at role or environment level).

#### Using an attribute as an indexer to another attribute
Using an attribute as an indexer to another attribute as per the example below

```
# attributes/default.rb
# Declare and assign the attributes
node.default['Greeting']['am'] = "Good morning"
node.default['Greeting']['pm'] = "Good afternoon"
node.default['IsAmOrPm'] = "pm"
 
# Logic in the recipe to assign a value to the 'IsAmOrPm' attribute
if Time.now.hour < 12 then
    node.default['IsAmOrPm'] = "am"
end
 
# recipe/default.rb
# Using the 'IsAmOrPm' attribute to refer to the nested attribute beneath 'Greeting'
log "LogGreeting" do
  message "#{node['Greeting'][node['IsAmOrPm']]} Chef client is now running ..."
  level :info
end
```

Is generally NOT recommended because changing an attribute that is an indexer does not give any indication of an impact; however in certain circumstances using a attribute as an indexer is the best method, such as selecting a version of software to install when a cookbook supports multiple versions.

### Miscellaneous 
Prefer attributes over local variables for settings calculated at runtime as these can be viewed by querying the Chef server when the run has completed. The exceptions are of course values such as secrets or keys. If you NEED to store values in attributes for the duration of the Chef run but do not want them sent back to the Chef server then consider using Blacklists or Whitelists or node.run_state.

Try and keep all attribute value modifications in one location, either an attribute file or recipe file, distributing the attribute assignment across several recipes is not recommended, where possible declare and assign the attribute in the attributes file.

Apply caution when assigning an attribute a value of hash or array, if using an array then individual array elements cannot be changed the entire attribute value has to be replaced with a new array, if using a hash the order of items in a hash cannot be guaranteed (although Ruby 1.9 has hashes ordered by insertion).

Preferred syntax for access an attribute is to use single quotes, double quotes imply that you will want to perform string interpolation

```
# Syntax for accessing attributes
node[:Greeting]
node["Greeting"]
node['Greeting']
 
# Preferred syntax is single quotes
node['Greeting']
```

### Nested attributes (and wrapper cookbooks)
Be careful when setting an attribute that is derived from another attribute, i.e. node['Greeting'] = "Hello #{node['User_Name']}", the attribute that you are deriving from may not have the correct value, it is recommended that you do not use nested attributes 

```
# A derived attribute value is a value that is constructed of other attribute values
######################################################
# Example 1.
######################################################
#Prefer
default['YourCompany']['URL'] = "http://localhost/products"
 
#Instead of
default['YourCompany']['URL'] = "http://#{node['YourCompany']['URL']}/products"
 
######################################################
# Example 2.
######################################################
#Prefer
default['YourCompany']['PaperTrailInstallFile'] = 'C:\\installers\\nxlog-ce-2.8.1248.msi'
 
#Instead of
default['YourCompany']['InstallationFolder'] = 'C:\\installers'
default['YourCompany']['PaperTrailInstallFile'] = "#{node['YourCompany']['InstallationFolder']}\\nxlog-ce-2.8.1248.msi"
```

An attribute that is built up from the value of another attribute might look a flexible way of setting attribute values but there is really no discernible advantage; it's far easier to debug the recipes when explicit values are used rather than work through derived values.

If you need REALLY need to embed attributes inside of other attributes then use string interpolation with named parameters
```
# Prefer
# Attributes file
default['version'] = '1.0'
default['url'] = "http://example.com/%{version}.zip"
 
# Recipe file
full_url = node['url'] % {version: node['version']}
 
# Over
# Attributes file
default['version'] = '1.0'
default['url'] = "http://example.com/#{node['version']}.zip"
```

Don't foget that attributes in attributes files of dependant cookbooks are evaluated before the wrapping cookbook; this makes it easy to override attribute values in dependant cookbooks but if the cookbook being wrapped uses nested attributes it can produce unexpected results.

### Store full values in attributes
Should your cookbook require 'sub component' values from an attribute use the fully qualified details to to derive the sub components; for example if you need a path element and filename element store the full path and filename in the attribute and use Ruby to extract the elements that you want.

```
### Sub elements from Attribute 
######################################################
# Attributes file
######################################################
default['YourCompany']['InstallationFile'] = 'C:\\installers\\nxlog-ce-2.8.1248.msi'
 
######################################################
# Recipe file
######################################################
# Get the filename (nxlog-ce-2.8.1248.msi)
nxlog_filename = filename(node['YourCompanyu']['InstallationFile'])
# Get the path (C:\\Installers\\)
nxlog_folder = folder(node['YourCompany']['InstallationFile'])


directory "createInstallersFolder" do
	path nxlog_folder
	recursive true
end
 
######################################################
# Helper library
######################################################
require 'pathname'
require 'uri'

# File.basename if using simpler scenarios
def filename(uri)
  Pathname.new(URI.parse(uri).path).basename.to_s
end

# File.dirname if using simpler scenarios
def folder(uri)
  Pathname.new(URI.parse(uri).path).dirname.to_s
end
```

### Attribute logic
Prefer to apply conditional logic in a Cookbook attribute file and store the result of the logic as an attribute value in order to make the cookbook and recipes simpler to read.

```
# default.rb attribute file from the Apache2 cookbook
case platform
when "redhat", "centos", "scientific", "fedora", "suse", "amazon", "oracle"
  default['apache']['package'] = "httpd"
  default['apache']['dir']     = "/etc/httpd"
  default['apache']['log_dir'] = "/var/log/httpd"
when "debian", "ubuntu"
  default['apache']['package'] = "apache2"
  default['apache']['dir']     = "/etc/apache2"
  default['apache']['log_dir'] = "/var/log/apache2"
when "arch"
  default['apache']['package'] = "apache"
  default['apache']['dir']     = "/etc/httpd"
  default['apache']['log_dir'] = "/var/log/httpd"
when "freebsd"
  default['apache']['package'] = "apache22"
  default['apache']['dir']     = "/usr/local/etc/apache22"
  default['apache']['log_dir'] = "/var/log"
else
  default['apache']['dir']     = "/etc/apache2"
  default['apache']['log_dir'] = "/var/log/apache2"
  default['apache']['error_log'] = "error.log"
end
 
# default.rb recipe of the Apache2 cookbook, simple to read as it just deals with attributes, no complex or confusing logic
package "apache2" do
  package_name node['apache']['package']
end
```

## Chef taking the strain
Look for inbuilt Chef functions that will simplify setting attributes such as platform_specific_path.
```
# Work out the correct path dependant on the O/S
case node['platform_family']
when "rhel", "fedora", "suse"
	default['YourCompany']['somepath'] = '/etc/chef/databag_secret'
when "windows"
	default['YourCompany']['somepath'] = 'c:/chef/databag_secret'
end
 
# Above can be replaced with a Helper method or built in Chef function
default['YourCompany']['somepath'] = Chef::Config.platform_specific_path('/etc/chef/databag_secret')
```
