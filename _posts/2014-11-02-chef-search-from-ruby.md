---
title: "Chef: Search from Ruby"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
  - ruby
---

Chef runs on Ruby but if your unfamiliar with Ruby then it's difficult to work out what syntax relates to Chef and what syntax is inherited from Ruby.
<!--more-->
It's taken a while but I'm managing to slowly prise apart Chef and Ruby in the hope that I'll gain a better understanding of both.

To this effect I saw this really good article about [abusing the chef api](http://erik.hollensbe.org/2012/02/24/abusing-the-chef-api-for-fun-and-profit/), the blog title is unusual but the content demonstrate that Chef code is basically Ruby.

The blog in question shows how to run a Chef search use pure Ruby, this is something that ought to be quite simple but Chef hides it's underlying Ruby classes and when used in a Cookbook automatically provides client configuration.

The code in the blog is below, I've added comments
```ruby
#!ruby

# Include the following Ruby files
require 'rubygems'
require 'chef/rest'
require 'chef/search/query'

# Load in the Chef configuration
Chef::Config.from_file(File.expand_path("~/.chef/knife.rb"))
# Create an instance of the Chef search object
query = Chef::Search::Query.new
# Run the query
nodes = query.search('node', ARGV.shift).first rescue []
# Output the query results
puts nodes.map(&:name).join("\n")
```

Some of the Ruby is a bit advanced, especially for someone new to the language but essentially the code
- Provide Chef with configuration (which will already be present on the client in the knife.rb file).
- Create an instance of the underlying Chef search class
- Grab the FIRST item back from the results list, on failure do nothing
- Take the resule and extract the :name attribute
- Append new line to the output

Distilling the code further to the bare bones we get
```ruby
require 'rubygems'
# Below is including all important chef libraries, could just require 'chef/rest' and require 'chef/search/query'
require 'chef'  

# Chef configuration
Chef::Config[:node_name]='chris_sullivan'
Chef::Config[:client_key]='C:/Users/chris.sullivan/.chef/chris_sullivan.pem'
Chef::Config[:chef_server_url]='https://chef.io'     


# Select some nodes
query = Chef::Search::Query.new
nodes = query.search('node', query="name:chris*")
nodeData.each do |node|  
	puts node 
end
```

![01-chef-search-from-ruby]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-11-02-chef-search-from-ruby/01-chef-search-from-ruby.jpg){: .align-center}

So although I hadn't initially understood the relevance of the "nodes = query.search('node', ARGV.shift).first rescue []" statement in the original code I realised that the search returns multiple collections. To find out what each node item contained I used the Ruby "inspect" keyword 

```ruby
nodes.each do |node|
	# Reflect on the node to see what is available
	puts node.inspect.to_s
end
```

Revising the code we can extract the main search results (the nodes) and iterate through them (NB: the code requires better exception handling!).

```ruby
require 'rubygems'
require 'chef'  


# Also seen Chef::Config.from_file(File.expand_path("~/chef/kinfe.rb")) - Or Env[]
Chef::Config[:node_name]='chris_sullivan'
Chef::Config[:client_key]='C:/Users/chris.sullivan/.chef/chris_sullivan.pem'
Chef::Config[:chef_server_url]='https://chef.io'     


# Select some nodes
query = Chef::Search::Query.new
nodes = query.search('node', query="name:chris*")
# Could also specify "first"
nodeData = nodes[0]
nodeData.each do |node|  
	puts node.name
end
```

![02-chef-search-from-ruby-node-attribute]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-11-02-chef-search-from-ruby/02-chef-search-from-ruby-node-attribute.jpg){: .align-center}

What is cool is that now we have setup Chef with the appropriate configuration we have access to all of the Chef goodness such as the Node class

```ruby
require 'rubygems'
require 'chef'  


Chef::Config[:node_name]='chris_sullivan'
Chef::Config[:client_key]='C:/Users/chris.sullivan/.chef/chris_sullivan.pem'
Chef::Config[:chef_server_url]='https://chef.io'     


Chef::Node.list.each do |node|  
    puts node.first      
end
```

Say we want to use Ruby to script our Chef queries can we bring in an library which is smaller than using the whole Chef library? The answer is yes and here are some alternatives.

### Ridley
[Ridley](https://github.com/reset/ridley)) is a nicely packaged gem that will allow you to access Chef resources using the Chef API from Ruby code.

The Ridley GitHub page is adorned with some very nice example code, within a few minutes I had various examples up and running

```ruby
require 'ridley'


# Below is required otherwise exception is thrown, see https://github.com/reset/ridley/issues/293
Ridley::Logging.logger.level = Logger.const_get 'ERROR'


# Example 1. Create an instance of Ridley, display role and node objects (may just display class name and memory location)
ridley = Ridley.new(
  server_url: 'https://chef.io',
  client_name: 'chris_sullivan',
  client_key: 'C:/Users/chris.sullivan/.chef/chris_sullivan.pem'
)
ridley.role.all
ridley.node.all


# Example 2. Create an instance of Ridley from knife configuration file, display all Roles
ridleyFromConfig = Ridley.from_chef_config('C:/Users/chris.sullivan/.chef/knife.rb')
ridleyFromConfig.role.all.each do |x|
	puts "Role is #{x.name} - #{x.description}"
end


# Example 3. Create an instance of Ridley for one off use, connection will be closed at the end of the block 
Ridley.open(server_url: "https://chef.io", client_name: 'chris_sullivan', client_key: 'C:\Users\chris.sullivan\.chef\chris_sullivan.pem') do |r|
  puts "Showing all nodes #{r.node.all}"
end

# Example 4. Use Ridley to display data
Ridley.open(server_url: "https://chef.io", client_name: 'chris_sullivan', client_key: 'C:\Users\chris.sullivan\.chef\chris_sullivan.pem') do |r|
  # Show all environments
  puts "Showing all environments #{r.environment.all}"
  # Get individual environment
  dbServerProductionCloud11 = r.environment.find("DatabaseServer-Production-Cloud11")
  puts dbServerProductionCloud11
  
  # Search for environments
  dbEnvrionments = r.search(:environment, "name:DatabaseServer*")
  dbEnvrionments.each do |e|
     puts e.name
  end
  
  # Alternative to using search
  filteredEnvironments = r.environment.all.select{|x| x.name.start_with?("DatabaseServer")}
  filteredEnvironments.each do |e|
    puts e.name
  end
end
```

Ridley will use all of the Chef API commands including PUT and DELETE so server update is possible.

### Chef-API
Cut down version of Ridley is [chef-api](https://github.com/sethvargo/chef-api) developed by Seth Vargo.
Chef-API is similar in approach to Ridley and is simple to use.

When you are first immersed into the Chef ecosystem it is difficult to understand what is going on but over time you can peel back many beautiful coloured layers.