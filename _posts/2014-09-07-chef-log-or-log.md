---
title: "Chef: Chef::log or log resource"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

At some point in your Chef development you'll want to send some output to the Chef log, if this is the case you are presented with two choices, Chef::Log or log resource. 
<!--more-->
Given the following code in a recipe, which text would you expect to be written to the log output first?
```ruby
log "showMessageOne" do
 message "****** Hello World"
 level :info
 action :write
end

Chef::Log.info("------- Bonjour!")
```

The actual output is below

![01-log-resource-log-class]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-09-07-chef-log-or-log/01-log-resource-log-class.jpg){: .align-center}

The second log message is being output first so what is going on?

Well there is no functional difference between the two; they will both log to Chef’s logger (Console Windows or log file like /var/log/chef/client.log on *nix).

As Chef resource requires two components, the resource and provider, navigating to the Chef installation folder we can see that the log resource does use the Chef::Log

Below is the log resource
```ruby
class Chef
  class Resource
    class Log < Chef::Resource
      identity_attr :message
      # === Parameters
      # name<String>:: Message to log
      # collection<Array>:: Collection of included recipes
      # node<Chef::Node>:: Node where resource will be used
      def initialize(name, run_context=nil)
        super
        @resource_name = :log
        @level = :info
        @action = :write
        @allowed_actions.push(:write)
        @message = name
      end
      def message(arg=nil)
        set_or_return(:message, arg, :kind_of => String)
      end
      # <Symbol> Log level, one of :debug, :info, :warn, :error or :fatal
      def level(arg=nil)
        set_or_return(:level, arg, :equal_to => [ :debug, :info, :warn, :error, :fatal ])
      end
    end
  end
end
```

and here is the provider
```ruby
class Chef
  class Provider
    class Log
      # Chef log provider, allows logging to chef's logs from recipes
      class ChefLog < Chef::Provider
        # ordered array of the log levels
        @@levels = [ :debug, :info, :warn, :error, :fatal ]
        # No concept of a 'current' resource for logs, this is a no-op
        #
        # === Return
        # true:: Always return true
        def load_current_resource
          true
        end
        # Write the log to Chef's log
        #
        # === Return
        # true:: Always return true
        def action_write
          Chef::Log.send(@new_resource.level, @new_resource.message)
          # resolve the integers for the current log levels
          global_level = Mixlib::Log::LEVELS.fetch(Chef::Log.level)
          resource_level = Mixlib::Log::LEVELS.fetch(@new_resource.level)
          # If the resource level is greater than or the same os the global
          # level, then it should have been written to the log.  Mark the
          # resource as updated.
          if resource_level >= global_level
            @new_resource.updated_by_last_action(true)
          end
        end
      end
    end
  end
end
```

## Compile and Converge
The reason for the Chef::Log to be output first is actually to do with the way Chef runs.

Chef has two phases of execution; a compilation phase and an execution phase. In the compilation phase, the recipes are evaluated as Ruby code, and resources are added to the resource collection. Next, is the execution phase (also called the converge), Chef runs the appropriate provider’s action on each resource.

Using Chef::Log; is the low level Chef code (Ruby) which means that log statements don’t actually create a resource in the resource collection. The log resource will always register as a resource that is updated; in other words, the :write action always runs. Using the log resource means that the action is not carried out until the second phase of execution.

However you can coerce the log resource to execute early
```ruby
log "showMessageOne" do
	message "****** Hello World"
	level :info
	action :nothing
end.run_action(:write)

e = log "showMessageTwo" do
	message "****** Guten Tag"
	level :info
	action :nothing
end
e.run_action(:write)

Chef::Log.info("------- Bonjour!")
```

Output is below

![02-log-resource-log-class-immediate-execution.jpg]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-09-07-chef-log-or-log/02-log-resource-log-class-immediate-execution.jpg){: .align-center}

Although I do NOT recommend running a resource in the first execution phase, the second execution phase is more complex than it first appears and if executing a resource in the first phase Subscribe and Notify eventing mechanisms will not work.

Also if you don't specify an action of :nothing or you specify the action more than once it will still register the resource and carry out the action in phase two.
```ruby
log "showMessageOne" do
	message "****** Hello World"
	level :info
	action :write
end.run_action(:write)

e = log "showMessageTwo" do
	message "****** Guten Tag"
	level :info
	action :write
end
e.run_action(:write)


Chef::Log.info("------- Bonjour!")
```

Output is

![03-log-resource-log-class-multiple-actions.jpg]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-09-07-chef-log-or-log/03-log-resource-log-class-multiple-actions.jpg){: .align-center}

So in summary, if you want to log during phase one then use Chef::Log and if you want to log during phase two then use the log resource.