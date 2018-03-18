---
title: "VirtualBox: Installing Ubuntu Desktop"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - VirtualBox
  - Ubuntu
---

This article could have been called "VirtualBox: Installing Ubuntu the naive way" because as I've found out there is more to think about when installing Ubuntu on a virtual machine if you are planning on keeping it around for a while.

Typically my omissions included
- considering where applications should be installed
- what happens when the root file system fills up
- swap file, swap partition or neither
<!--more-->

If you do want to install Ubuntu for short term use or just want to experiment then here the steps for creating an Ubuntu virtual machine; VirtualBox does guide you through this process with a series of dialogs.

## Virtual Machine
The purpose of this article is to install Ubuntu Desktop in a [Virtual Machine (VM)](https://en.wikipedia.org/wiki/Virtual_machine); the software to run Virtual Machines that I have chosen is [VirtualBox](https://www.virtualbox.org/), it is free and robust. VirtualBox can be downloaded at [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads).

Just to clarify the machine running VirtualBox (or the VM software) such as your desktop/laptop or other is called the host and the Virtual Machine itself is the guest.

The process for installing VirtualBox on the host is easy and once installed we are good to go.

When following the steps there are certain assumptions made that you will have basic knowledge or experience of VirtualBox or another software package that can run Virtual Machines.

## Step 01. Create a new Virtual Machine
Open the VirtualBox manager, on most operating systems you can search the applications.

In Windows click on the start icon and start typing "Virtual" or open a command prompt just type "virtualbox" and press return, the VirtualBox manager application should be in the command path.

When the VirtualBox manager has opened click on the New icon or hold down Ctrl+N or select Machine from the menu item and choose New.

![01-new-virtual-machine]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/01-new-virtual-machine.png){: .align-center}

The Create Virtual Machine dialog will appear; give your VM a name, the type is Linux and verion is Ubuntu (64-bit)

![02-name-type-version]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/02-name-type-version.png){: .align-center}

Click next to proceed.

## Step 02. Set the virtual machine memory size

Ubuntu is a nice lean operating system and doesn't require much memory in order to perform basic tasks.

![03-virtualbox-memory]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/03-virtualbox-memory.png){: .align-center}

NB:
- the virtual machine will use the physical memory of your host machine
- you should leave enough memory for the host to function properly
- consider if you want to run multiple virtual machines at the same time
- with our Ubuntu install a swap partition equaling the size of the memory will be created

Click next to proceed.

## Step 03. Create your hard disk
Ubuntu needs to be installed on a hard disk so we need to be able to create one

![04-select-hard-disk]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/04-select-hard-disk.png){: .align-center}

After selecting create we need to choose the hard disk file type.

![05-select-disk-file-type]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/05-select-disk-file-type.png){: .align-center}

The Virtual Machine hard disk exists as a file or files on the host machine. The internal format of that file can be vendor specific locking your VM into only being able to run with one type of software. Fortunately VirtualBox supports the following Virtual Machine [file formats](https://www.virtualbox.org/manual/ch05.html#vdidetails)
### [VDI](https://en.wikipedia.org/wiki/VirtualBox#VirtualBox_Disk_Image)
VDI is the native format of VirtualBox. 

### [VMDK](https://en.wikipedia.org/wiki/VMDK)
VMDK is developed by and for VMWare, but Sun xVM, QEMU, VirtualBox, SUSE Studio, and .NET DiscUtils also support it.

### [VHD](https://en.wikipedia.org/wiki/VHD_(file_format))
VHD is the native format of Microsoft Virtual PC.

### HDD
HDD is Parallels version 2 format.

VDI, VMDK, and VHD all support dynamically allocated sizing. VMDK has an additional capability of splitting the storage file into files less than 2 GB each, which is useful if your file system has a small file size limit.

For the purposes of this article we are going native and selecting VDI; if we need to change the software running the Virtual Machine then VirtualBox allows us to export the disk to an [Open Virtualisation Format](https://en.wikipedia.org/wiki/Open_Virtualization_Format) so that can be imported by another product.

Click next to proceed.

Deciding on how much hard disk space you require depends on what you want your Virtual Machine to do; VirtualBox allows you to specify a fixed disk size or dynamically allocated. If you select dynamically allocated disk then VirtualBox will expand the disk as required, this is useful if you do not know how much space you will need to start with, don't worry though if you select fixed size you can manually expand the drive later.

![06-storage-on-physical-disk]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/06-storage-on-physical-disk.png){: .align-center}

Note:
- the virtual machine will use the physical drive space of your host machine so make sure you have enough
- with fixed or dynamic disks you can resize at a later date

Specify a disk size.

![07-file-location-size]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/07-file-location-size.png){: .align-center}

With our Ubuntu install the drive size needs to include a swap partition equalling the size of the memory allocated to the Virtual Machine, therefore if you have assigned 2Gb of memory and require 6Gb of usable space the drive size needs to be at least 8Gb.

Click create to proceed.

## Step 4. Install Ubuntu
With an empty disk created we are now ready to [install Ubuntu desktop](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop#0)

The empty disk will appear in the Virtual Box Manager 

![08-empty-disk-created]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/08-empty-disk-created.png){: .align-center}

and from now on we will call it an appliance.

Installing Ubuntu is reasonably easy, we will download a disc image in ISO format and install one of the Long Term Support (LTS) versions [14.04](https://www.ubuntu.com/download/desktop). Keep hold of this downloaded image as it acts as a "Live CD" and will allow us to modify the installation ar a later date.

Double click appliance, the select start-up disk dialog will open, click on the folder and locate the ISO, I'm installing 14.04.5 LTS

![09-select-startup-disk]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/09-select-startup-disk.png){: .align-center}

Click start to proceed.

The Ubuntu disc will be inserted into a virtual drive and Ubuntu will boot.

![10-ubuntu-try-install]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/10-ubuntu-try-install.png){: .align-center}

Select install Ubuntu, the prepare to install Ubuntu screen will display along with any warnings or recommendations

![11-preparing-install-ubuntu]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/11-preparing-install-ubuntu.png){: .align-center}

Click continue to proceed.

The installation process allows you customise your installation, we are performing a basic installation here, the defaults are all good.

![12-installation-type]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/12-installation-type.png){: .align-center}

Click Install Now to proceed. Ubuntu will confirm the drive settings that we have selected.

![13-write-changes-to-disks]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/13-write-changes-to-disks.png){: .align-center}

Click continue to proceed. Ubuntu now confirms your timezone locations

![14-where-are-you]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/14-where-are-you.png){: .align-center}

Click continue to proceed. Select your keyboard layout

![15-keyboard-layout]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/15-keyboard-layout.png){: .align-center}

Click continue to proceed. Enter the computer name, your name and password, these details will be used to login when the Ubuntu installation is complete.

![16-who-are-you]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/16-who-are-you.png){: .align-center}

Click continue to complete the setup. It doesn't take too long for Ubuntu to install

![17-installing-welcome-ubuntu]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/17-installing-welcome-ubuntu.png){: .align-center}

When the disk has been prepared the installer will reboot

![18-installation-complete-restart]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/18-installation-complete-restart.png){: .align-center}

If you selected automatic login Ubuntu will log you in ready to start your desktop session, if you did not select automatic login then enter your username and password

![19-login-to-ubuntu]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/19-login-to-ubuntu.png){: .align-center}

After logging in you are presented with the Ubuntu desktop

![20-ubuntu-desktop]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/20-ubuntu-desktop.png){: .align-center}

## Step 05. VirtualBox guest additions
For optimal performance we need to install the VirtualBox guest additions on Ubuntu; this enables better integration between guest and host operating systems and permits the use of shared clipboard and drag and drop. The Guest Additions CD matches the version of VirtualBox and is supplied as part of the VirtualBox installation on your host operating system.

### Install Guest Additions
From the VirtualBox menu select Devices -> Install Guest Additions CD image

![21-install-guest-additions]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/21-install-guest-additions.png){: .align-center}

To install the software you will need super user permissions; the user you created has super user permissions so enter the password you have used to login

![22-installation-authentication]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/22-installation-authentication.png){: .align-center}

Guest additions will proceed to install and will require a reboot

![23-additions-installed-restart]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/23-additions-installed-restart.png){: .align-center}

### Enable features
When the guest operating system is running click on the menu and select
Machine -> Settings -> General -> Advanced -> enable Shared Clipboard and/or Drag'n'Drop
Make sure View -> Auto-resize Guest Display is checked

![24-virtualbox-guest-host-features
]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-10-23-virtualbox-install-ubuntu/24-virtualbox-guest-host-features.png){: .align-center}

## Summary
Now you can start to do [fun and cool stuff](https://tutorials.ubuntu.com/) with one of the best Linux distributions available.

But I'll finish with a brief explanation of Open Virtualisation Formats

### [Open Virtualization Format (OVF)](https://en.wikipedia.org/wiki/Open_Virtualization_Format)
The OVF Specification provides a means of describing the properties of a virtual system. It is XML based and has generous allowances for extensibility (with corresponding tradeoffs in actual portability). Most commonly, an OVF file is used to describe a single virtual machine or virtual appliance. It can contain information about the format of a virtual disk image file as well as a description of the virtual hardware that should be emulated to run the OS or application contained on such a disk image.

### Open Virtual Appliance (OVA)
An OVA is an OVF file packaged together with all of its supporting files (disk images, etc.). You can read about the requirements for a valid OVA package in the OVF specification. Oftentimes people will say “an OVF” and really mean “an OVA.”

