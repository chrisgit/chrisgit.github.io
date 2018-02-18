---
title: "DevOps: London Meetup"
excerpt: "Having spent a few weeks working with Chef and attempting to build the infrastructure at speed on Windows I wanted to get the views of other similar professionals using similar tooling."
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - devops
  - chef
  - meetups
---

A few weeks ago I attended an Infrastructure Automation on Windows meetup in the newly decorated JustGiving Head Office, the meetup was recorded and can be found [here](https://vimeo.com/channels/londoncd/107488886).

All of the speakers were very good at conveying their stories of using Automation tools in Windows, some of the key take away points are
- We are not alone in using the latest provisioning tools (arguably being part of the bleeding edge for Automation)
- Automating on Windows is (currently) harder and takes longer to develop than our *NIX counterparts

One of the highlights of the evening was a talk by Stephen Nelson-Smith author of Test-Driven Infrastructure with Chef; if you want to see Stephens talk it starts around 35th minute on the linked video, the content below is a summary of his presentation with some additional content.

## Configuration Management
![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-09-18-london-meetup/evolution-computers.jpg){: .align-center}

“Configuration management maturity is a pre-requisite for organisational effectiveness”

When considering the evolution of Configuration management we can categorise the typical progress in five levels

### Level 1
Hand crafted manually built appliances, network contains snowflakes, there is little or no shared knowledge.

A snowflake is used to identify that each appliance in the network contains variances even if they provide the same role.

### Level 2
Some documentation exists, Wiki articles, gold images (possibly old or out of date), basic scripting (possibly using perl)

### Level 3
Early first and second generation management tools such as CF Engine and early Puppet, some use of version control, the elements of being declarative but just a step beyond level 2

### Level 4
Third or fourth generation management tools, element of desired state, declarative interface, searching capability, able to pull data, handle multiple sites, most current tools such as SALT, Ansible, Chef and Puppet

### Level 5
Everything in level 4 plus orchestrations, continuous delivery, high level auditing and reporting, some analytics

Before the company I work for started to improve their infrastructure automation it was fair to say we would best be described at Level 2 this is partly due to the lack of a simple to use scripting model for our Windows servers.

Microsoft had some attempts at building a scripting language introducing the Windows Script Host (cscript followed by wscript). Windows Script Host integrates with Active Script Engine and allows scripts to be written in JScript and VBScript which call into API’s via COM.

Therein lies the problem, while *NIX operating system has a good collection of scripting tools these work because they are based against operations on files and streams, Windows commands operate against API’s.

In 2002 Microsoft started a project call Monad; Monad was to be a new extensible command shell, the first beta was released in 2005 and the full product, now renamed to PowerShell was released in 2006.

## Desired State Configuration

![image-center]({{ site.url }}{{ site.baseurl }}/assets/posts/2014-09-18-london-meetup/jlp-make-it-so.jpg){: .align-center}

Microsoft are describing Desired State Configuration as “Make It So”, what that really means is carry out some actions on a server and place
it in a known state, the desired state.

Desired State Configuration (or DSC) is a PowerShell extension and simplifies the following

- Install or remove server roles and features
- Manage registry settings
- Manage files and directories
- Start, stop, and manage processes and services
- Manage local groups and user accounts
- Install and manage packages such as .msi and .exe
- Manage environment variables
- Run Windows PowerShell scripts
- Fix a configuration that has drifted away from the desired state
- Discover the actual configuration state on a given node

## Chef support for DSC

Chef is an excellent provisioning tool, out of the box Chef supports deployment onto *NIX and Windows machines providing support
- installing packages
- running shell scripts (inc. PowerShell on Windows)
- interrogating the appliance

As of version 11.16.0 Chef now supports Microsoft DSC with a new [dsc_script](https://blog.chef.io/2014/09/08/chef-11-16-gets-into-powershell-dsc/) resource.

## Future of provisioning on Windows machines

With the advent of Windows 2012 headless mode (no Windows GUI) and the continued success of Azure (no doubt helped by promoting [specialist facilities](https://msdn.microsoft.com/en-us/magazine/dn532200.aspx)) the future of Windows management and automation has to become simpler.

Microsoft are making headway into improving and building better package managers to install your software with [Oneget](https://github.com/OneGet/oneget) (due to be shipped with Windows 10), which will fully support [Chocolately](https://chocolatey.org/), alternatively why not combine Chocolatey and package your installers with [Boxstarter](http://boxstarter.org/).

While Linux ops guys have SSH the preferred way of Connecting to Microsoft servers is via WinRM (Windows Remote Management) which is a SOAP based web service that can send and receive messages via HTTP(s) and translate them to the local command line. PowerShell has it's own communication mechanism called [PowerShell Remoting](https://technet.microsoft.com/en-us/library/ff700227.aspx) that sits on top of WinRM but the technology is not limited to the Microsoft stack, WinRM is fully supported in Ruby (as a downloadable [Gem](https://rubygems.org/gems/winrm)).  Talking of PowerShell it is becoming more powerful and widespread, filling the gaps of missing tools and API's on the Windows platform, the freshly developed desired state configuration (DSC) being some of the latest enhancements, the latest release [wave 7](https://blogs.msdn.microsoft.com/powershell/2014/09/26/continuing-the-dsc-resource-kit-additions-wave-7-is-live/) being only a few months old.

Provisioning and testing to Microsoft servers is set to become quicker and easy with containers, early examples come from [Spoon.net](https://turbo.net/) but Microsoft themselves want to [provide this functionality](https://azure.microsoft.com/en-us/blog/new-windows-server-containers-and-azure-support-for-docker/).

The good news is that OpsCode Chef is reacting quickly to these changes meaning we can continue to improve and develop our cookbooks using the recommended technology for the job.

## How do I get PowerShell 4.0 / DSC

Below is an extract from [how to install PowerShell 4.0](http://social.technet.microsoft.com/wiki/contents/articles/21016.how-to-install-windows-powershell-4-0.aspx)

### How to Install Windows PowerShell 4.

Windows PowerShell 4.0 is part of the Windows Management Framework 4.0, which includes the following:

- Windows PowerShell
- Windows PowerShell Integrated Scripting Environment (ISE)
- Windows PowerShell Web Services (Management OData IIS Extension)
- Windows Remote Management (WinRM)
- Windows Management Infrastructure (WMI)
- Server Manager WMI provider
- Windows PowerShell Desired State Configuration (DSC)

### Windows Management Framework 4.0 supportability matrix

PowerShell 4.0 is built in to Windows Server 2012 R2 and Windows 8.1

Windows Server 2012, Windows Server 2008 R2 and Windows 7 PowerShell 4.0 is available as part of WMF. For Windows Server 2008 R2 and Windows 7 .Net 4.5 will need to be installed first, Windows Server 2012 has .Net 4.5 built in.

Windows 8 users need to upgrade to 8.1.

#### Installation

Verify that all prerequisites are installed according to the Windows Management Framework 4.0 supportability matrix above. To verify the presence of .NET 4.5, you may use the Test-Net45 on the [Hey Scripting Guy! Blog](https://blogs.technet.microsoft.com/heyscriptingguy/2013/11/02/powertip-find-if-computer-has-net-framework-4-5/)

Run the installation file applicable to the operating system. Reboot the computer, start Windows PowerShell and verify that the output of $PSVersionTable shows 4.0 as the value of the PSVersion property

#### Known issues

Installation succeeds even if .NET 4.5 is not installed

Scenario: Installing WMF 4.0 on a computer that is not running .NET Framework 4.5 will report that the installation is successful, but the
components of WMF 4.0 (such as Windows PowerShell, WMI, etc.) will not be updated.

Solution: Install .NET Framework 4.5, and then run the WMF 4.0 installer again.

More information:
[http://blogs.msdn.com/b/powershell/archive/2013/10/29/wmf-4-0-known-issue-partial-installation-without-net-framework-4-5.aspx](http://blogs.msdn.com/b/powershell/archive/2013/10/29/wmf-4-0-known-issue-partial-installation-without-net-framework-4-5.aspx)

#### Compatibility issues

There are known compatibility issues with several Microsoft server-class applications:

- System Center 2012 Configuration Manager (not including SP1)
- System Center Virtual Machine Manager 2008 R2 (including SP1)
- Microsoft Exchange Server 2013, Microsoft Exchange Server 2010, and Microsoft Exchange Server 2007
- Microsoft SharePoint Server 2013 and Microsoft SharePoint Server 2010
- Windows Small Business Server 2011 Standard

Read the WMF 4.0 Release Notes for more information.

#### Related KB articles

Update is available that prevents the PSModulePath environment variable from being reset when you upgrade WMF 3.0 to WMF 4.0 and then uninstall WMF 4.0 in Windows [http://support.microsoft.com/kb/](http://support.microsoft.com/kb/)

Update prevents the “PSModulePath” environment variables from being reset after you uninstall WMF 4.0 in Windows
[http://support.microsoft.com/kb/](http://support.microsoft.com/kb/)

#### Update 08 Dec 2014

[PowerShell DSC Wave 8](http://blogs.msdn.com/b/powershell/archive/2014/10/28/powershell-dsc-reskit-wave-8-now-with-100-resources.aspx)

[All DSC packages](https://gallery.technet.microsoft.com/DSC-Resource-Kit-All-c449312d)
