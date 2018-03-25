---
title: "Chef: Attributes (part 4)"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
---

Chef attributes are complicated, knowing when to set them is not easy, this article is not intended to replace the excellent documentation found at the OpsCode web site but rather to compliment it.
<!--more-->

## Blacklisting and whitelisting attributes
After a chef converge the Chef client will send the node data (attributes) back to the Chef server. The node data consists of automatic attributes gathered by Ohai as well as all of the attributes used in Chef cookbooks.

Ohai is a Chef tool that runs a collection of plugins, the plugins gather information about the servers environment, for example the EC2 plugin will extract information about the servered based on knowing that it is an EC2 instance.

This is great but it can result in the Chef server storing a lot of attributes and not all of them are relevant to your day to day operations auditing.

### Whitelisting
Below image is from the [Chef.io attributes page](https://docs.chef.io/attributes.html#whitelist-attributes)

> Whitelist Attributes
> When these settings are used, any attribute not defined in a whitelist will not be saved. Each attribute type is whitelisted
> independently of the other attribute types. For example, if automatic_attribute_whitelist defines attributes to be saved, but
> normal_attribute_whitelist, default_attribute_whitelist, and override_attribute_whitelist are not defined, then all normal, default
> and override attributes are saved, along with only the specified automatic attributes.

Attributes that should be saved by a node may be whitelisted in the client.rb file. The whitelist is a Hash of keys that specify each attribute
to be saved.

Attribute are whitelisted by attribute type, with each attribute type being whitelisted independently. Each attribute type—automatic,
default, normal, and override—may define whitelists by using the following settings in the client.rb file:

#### automatic_attribute_whitelist 
A Hash that whitelists automatic attributes, preventing non-whitelisted attributes from being saved. For
example: ['network/interfaces/eth0']. Default value: all attributes are saved. If the Hash is empty, no
attributes are saved.

#### default_attribute_whitelist 
A Hash that whitelists default attributes, preventing non-whitelisted attributes from being saved. For
example: ['filesystem/dev/disk0s2/size']. Default value: all attributes are saved. If the Hash is empty, no
attributes are saved.

#### normal_attribute_whitelist
A Hash that whitelists normal attributes, preventing non-whitelisted attributes from being saved. For
example: ['filesystem/dev/disk0s2/size']. Default value: all attributes are saved. If the Hash is empty, no
attributes are saved.

#### override_attribute_whitelist
A Hash that whitelists override attributes, preventing non-whitelisted attributes from being saved. For
example: ['map - autohome/size']. Default value: all attributes are saved. If the Hash is empty, no
attributes are saved.

> It is recommended that only automatic_attribute_whitelist be used to whitelist attributes. This is primarily because automatic
> attributes generate the most data, but also that normal, default, and override attributes are typically much more important
> attributes and are more likely to cause issues if they are whitelisted incorrectly

For example, normal attribute data similar to:
```
{
  "filesystem" => {
    "/dev/disk0s2" => {
      "size" => "10mb"
    },
    "map - autohome" => {
      "size" => "10mb"
    }
  },
  "network" => {
    "interfaces" => {
      "eth0" => {...},
      "eth1" => {...},
    }
  }
}
```

To whitelist the network attributes and prevent the other attributes from being saved, update the client.rb file:
```
normal_attribute_whitelist ['network/interfaces/']
```

When a whitelist is defined, any attribute of that type that is not specified in that attribute whitelist will not be saved. So based on the
previous whitelist for normal attributes, the filesystem and map - autohome attributes will not be saved, but the network attributes will.

Leave the value empty to prevent all attributes of that attribute type from being saved:
```
normal_attribute_whitelist []
```

For attributes that contain slashes (/) within the attribute value, such as the filesystem attribute '/dev/diskos2', use an array. For example:
```
automatic_attribute_whitelist [['filesystem','/dev/diskos2']]
```

### Blacklisting
Blacklisting attributes is a popular request for the Chef team but unfortunately they have never got around to implementing the feature.
Update: Chef 13 now supports [blacklisting](https://docs.chef.io/attributes.html#blacklist-attributes)

If you want to disable attributes gathered by a particular Ohai plugin you can disable the plugin by changing the Chef client.rb configuration
file and disabling the plugin
```
Ohai::Config[:disabled_plugins] = ["plugin_to_be_disabled"]
```

If you want to selectively blacklist attributes you can add the following monkey patch (or write an appropriate handler)
```
class Chef
  class Node
    alias_method :old_save, :save
    def save
      self.default_attrs.delete(:key)
      self.normal_attrs.delete(:other_key)
      self.override_attrs.delete('...')
      self.automatic_attrs.delete('...')
      old_save
    end
  end
end
```

However, there is a community cookbook that will blacklist your attributes, see: https://github.com/irccloud/blacklist-node-attrs
