---
title: "VirtualBox: Resizing an Ubuntu Desktop partition"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - VirtualBox
  - Ubuntu
---

If you have installed your Ubuntu desktop VM in the most basic way; accepting the installer defaults and selected dynamically allocated disk you may be suprised to find out that it is running low on disk space.
<!--more--> 

Fortunately there is no guesswork in discovering low disk space; there is a helpful warning

![01-ubuntu-low-disk-space]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/01-ubuntu-low-disk-space.jpg){: .align-center}

Clicking examime displays where the spread of data resides

![02-ubuntu-examine-disk]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/02-ubuntu-examine-disk.jpg){: .align-center}

For anyone that prefers the command line then there is [df](https://en.wikipedia.org/wiki/Df_(Unix))
```
df 
```

or 
```
df --block-size=GB
```

![03-ubuntu-disk-space]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/03-ubuntu-disk-space.jpg){: .align-center}

Removing all those small files and duplications to claw back space is sometimes hardwork and slow, what we need is to resize the disk, while not difficult there are several steps to consider. To simplify the process we will use the GUI tool [GParted Partition Editor](https://gparted.org/).

First, shut down the Ubuntu desktop VM.

## Backup your existing disk
Before we increase the size of our virtual hard disk, it’s always a good idea to make a backup of it in case something go wrong.

Open the VirtualBox manager, select the Ubuntu desktop VM, click on the settings cog icon or select Machine from the menu and choose settings or hold down Ctrl+S.

When the settings dialog is open choose Storage, copy the location of your virtual hard disk.

![04-backup-get-location]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/04-backup-get-location.jpg){: .align-center}

Next open up a command window on your host OS and run the following command to backup the virtual hard disk.

```
cp /location-of-virtual-disk /location-of-backup-of-virtual-disk
```

## Increase Virtualbox Disk Size
The disk size needs to be increased, even if you are using a dynamically allocated disk.

Open a command window, vboxmanage is used to expand your virtual disk, the syntax is as follows

```
vboxmanage modifyhd /location-of-your-virtual-disk --resize size-in-MB
```

Specify the new size in Megabytes, a 1000 Megabytes is a Gigabyte.  To increase the size to ten Gigabytes the value is 10 * 1000 or  ten thousand.
```
vboxmanage modifyhd "/home/matrix/VirtualBox VMs/ubuntu/ubuntu.vdi" --resize 10240
```

The resize process is usually very quick and the sizing process displays progress status.

![05-resize-virtual-hard-disk]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/05-resize-virtual-hard-disk.jpg){: .align-center}

## Extend the Ubuntu partition

### Insert the Ubuntu Live CD
Open the VirtualBox manager, click on the settings cog icon or select Machine from the menu and choose settings or hold down Ctrl+S.

When the settings dialog is open choose Storage, select your optical drive; click on the picture of the Compact Disc; if you have already used the Ubuntu installation ISO image VirtualBox remembers and it is selectable otherwise click "Choose Virtual Optical Disk File..."

![06-insert-unbuntu-installation-disc]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/06-insert-unbuntu-installation-disc.jpg){: .align-center}

### Boot order
Check the boot order in VirtualBox to ensure that the Virtual Machine will boot from CD.

With the settings dialog open, select System and choose the Motherboard tab

![07-virtualbox-boot-order]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/07-virtualbox-boot-order.jpg){: .align-center}

The Optical drive should be the first boot device (or at least before the Hard Disk)

### Boot from the Live CD
You have to boot from the Live CD because otherwise the file system will be in use and GParted will not be able to make changes.

Double click the Ubuntu desktop VM to start it booting, when the Live CD has booted select Try Ubuntu.

![08-unbuntu-live-install-boot-screen]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/08-unbuntu-live-install-boot-screen.jpg){: .align-center}

### Open GParted Partition Editor
The Ubuntu desktop install comes with a GUI tool called GParted, click on Search your computer and online sources

![09-search-computer-online-services]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/09-search-computer-online-services.jpg){: .align-center}

Search for GParted

![10-search-gparted]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/10-search-gparted.jpg){: .align-center}

Double click the GParted Partition Editor icon.

The newly added disk space is shown in GParted as unallocated space at the end of the disk.

![11-gparted-unallocated-end]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/11-gparted-unallocated-end.jpg){: .align-center}

The unallocated space needs to be adjacent to the partition you are extending. In this case we want to extend /dev/sda1 and need to move the unallocated space next to /dev/sda1.

Moving the unallocated space reminds me of the [Towers Of Hanoi puzzle](https://en.wikipedia.org/wiki/Tower_of_Hanoi); here is presented two methods to extend the partition.

Increasing and decreasing the partition size with Gparted done graphically, right click the partition and select "Resize/Move"
- Grab the left arrow to shrink or expand
- Grab the right arrow to shrink or expand
- Grab the middle (white section) to move

The Ubuntu installation here has a swap partition which needs to be turned off; the key next to the partition name indicates that it is currently in use, right click on the linux-swap partition and select swapoff.

![12-turn-swap-off]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/12-turn-swap-off.jpg){: .align-center}

## Option 1. Without deleting the swap partition

### Summary of operations
To extend the /dev/sda1 partition
- Turn the swap partition off (right click swap off)
- Extend /dev/sda2 to include the unallocated space, grab right arrow, drag right
- Move /dev/sda5 (the swap partition) to the end of /dev/sda2, grab white section, drag right
- Reduce the size of /dev/sda2 to include only the swap /dev/sda5, grab left arrow, drag right

The unallocated space should now be adjacent to /dev/sda1
- Increase the size of /dev/sda1, grab right arrow, drag right
- Turn the swap file back on
- Reboot

## Detail of operations

First, resize your extended partition (not the one labeled linux-swap) to include the free space.

In this example select /dev/sda2, right click and choose Resize/Move. The Resize/Move dialog appears, drag the arrow on the right side of the bar to include all of the free space available and click on the Resize/Move button. 

An operation will appear at the bottom of the GParted application.

![13-gparted-operations]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/13-gparted-operations.jpg){: .align-center}

Gparted adds the commands to a queue, the commands will not execute until you select the green arrow to "Apply All Operations"; any wrong commands can be removed from the queue by selecting the orange arrow "Undo Last Operation".

The swap partition needs to be moved to be at the end of /dev/sda2. Select the linux-swap partition (/dev/sda5) and select Resize/Move. Inside the resize dialog left click the white space between the arrows and drag the box over to the right handside.

![14-move-the-swap-to-end]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/14-move-the-swap-to-end.jpg){: .align-center}

Note that moving a partition is particularly dangerous and is more liable to data loss or corruption. When a partition is moved, the files on that partition must be moved with it. Gparted issues a warning accordingly

![15-partition-move-warning]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/15-partition-move-warning.jpg){: .align-center}

The unallocated space now needs to be moved next to the main partition. Select /dev/sda2, "Resize/Move", move the left hand arrow to the right so that it sits on the edge of the swap partition.

![16-reduce-partition-sda2]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/16-reduce-partition-sda2.jpg){: .align-center}

The unallocated space is now next to the main partition /dev/sda1

![17-unallocated-next-sda1]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/17-unallocated-next-sda1.jpg){: .align-center}

Select /dev/sda1, "Resize/Move", grab the right hand arrow and drag it all the way to the right.

![18-sda1-resized-include-unallocated]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/18-sda1-resized-include-unallocated.jpg){: .align-center}

The operations are similar to the ones below

![19-gparted-operations-summary]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/19-gparted-operations-summary.jpg){: .align-center}

Click on the green arrow "Apply All Operations".

Right click the swap partition and select "Swapon", reboot your VM.

Check your disk sizes

![20-new-disk-size]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/20-new-disk-size.jpg){: .align-center}

## Option 2. Delete the swap partition

### Summary of operations
To extend the /dev/sda1 partition
- Turn the swap partition off (right click swap off)
- Delete the swap partition
- Extend /dev/sda2 to include the unallocated space, grab right arrow, drag right
- Recreate the linux swap partition and position at the end of /dev/sda2
- Reduce the size of /dev/sda2 to include only the swap /dev/sda5, grab left arrow, drag right

The unallocated space should now be adjacent to /dev/sda1
- Increase the size of /dev/sda1, grab right arrow, drag right
- Turn the swap file back on
- Reboot

### Detail of operations
Delete the current swap partition, right click on /dev/sda5, select delete

![21-delete-swap-file]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/21-delete-swap-file.jpg){: .align-center}

Increase the size of the /dev/sda2 partition, right click on /dev/sda2 and select "Resize/Move", grab the right hand arrow and drag it all the way to the end, the click on the "Resize/Move" button.

![22-increase-size-sda2]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/22-increase-size-sda2.jpg){: .align-center}

Recreate the swap partition. Right click on the unallocated space beneath /dev/sda2, select "Resize/Move", drag the left hand arrow to the right and ensure that the new size is the same as the old swap partition.

![23-create-new-swap-partition]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/23-create-new-swap-partition.jpg){: .align-center}
Note: make sure the file system is linux-swap.

Reduce the size of the /dev/sda2 partition, right click on /dev/sda2, select "Resize/Move", drag the left hand arrow to the right so it matches up with the swap partition, select the "Resize/Move" button.

![24-reduce=size-sda2]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/24-reduce=size-sda2.jpg){: .align-center}

The unallocated space is next to the partition that we want to extend, right click on /dev/sda1, select "Resize/Move", grab the right hand arrow and move it to the end, click on the "Resize/Move" button.

![25-increase-size-sda1]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/25-increase-size-sda1.jpg){: .align-center}

The operations are similar to the ones below

![26-gparted-operations-summary]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/26-gparted-operations-summary.jpg){: .align-center}

Click on the green arrow "Apply All Operations".

Right click the swap partition and select "Swapon", reboot your VM and login.

Click on "Search your computer and online resources", type in "System" and select System Monitor, click on the resources tab; the swap is not available.

![27-swap-not-available]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/27-swap-not-available.jpg){: .align-center}

The reason the swap is not available is becuase the swap partition has been given a new UUID.

There are a few ways to update the swap partition information; open a terminal window (Ctrl + Alt + T), get the current swap partition details

```
grep swap.*sw /etc/fstab
```

Check against the current partition details
```
sudo blkid | grep swap
```

If the UUIDs are different then edit /etc/fstab and change the swap UUID with the UUID that was returned by blkid.

Gparted partition editor will also show you the swap partition UUID, select the swap partition (/dev/sda5), right click and select "Information"

![28-swap-partition-uuid]({{ site.url }}{{ site.baseurl }}/assets/posts/2016-11-06-virtualbox-resize-ubuntu-partition/28-swap-partition-uuid.jpg){: .align-center}

After updating /etc/fstab Ubuntu desktop VM requires a restart.

When booted the new swap partition should be active.
