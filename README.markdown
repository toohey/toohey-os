# About toohey-os
This repository will contain releases of operating system that act as the starting point for building dev environment for Oracle Commerce. This is to ensure that different operating system release and versions can be used.

# Pre-requisites
* Virtual Box installed (tested on 4.3.3 on OSX )
* vagrant installed (tested on 1.7.4 on OSX)

# Usage
Download a specific release binary and use it. In terminal window, run
vagrant box add --name toohey-centos65 ADDRESS

# Licence
Each release licenced may be licenced under a different release

# Details - not needed to use
##Summary
This is essentially just a repackaging of minimal centos 6.5 based vagrant box found at 
https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
found it at http://www.vagrantbox.es search for centos 6.5

This creates a virtual machine with HDD size of 8192 MB. This proved to not be enough for our purposes. hence needed to resize the disks to 100 GB.

Resizing an existing virtual machine requires following steps (based on http://serverfault.com/questions/509468/how-to-extend-an-ext4-partition-and-filesystem):

* resize the HDD - sort of like magically adding more space of existing drive
* update partition information
* extend the filesystem

All need to be done without destroying data.

Finally, package the resized virtual machine release it here as a box.

##ToDo's
* automate this such that vagrant box can be created from .iso files directly

##Detailed Instructions
###create a virtual machine
why?: HDD has to exist before it can be resized  

	vagrant box add --name centos65   https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box  
	mkdir repack-centos65 #called **projdir** in rest of doc  
	cd repack-centos65  
	vagrant init centos65
	vagrant up
	vagrant halt

###Resize the disk
Virtual box creates actual machine images in a different place than projdir. 
find it. usually '~/VirtualBoxVms'. Mine is /Users/toohey/virtualMachines/VirtualBoxVms. 

> cd /Users/toohey/virtualMachines/VirtualBoxVms

this should have box-disk1.vmdk and box-disk2.vmdk
If you have Vmware tools, just extend the size using
vmware-vdiskmanager -x 100Gb vm.vmdk

***OR***

If you dont have VMware, use vboxmanage from virtual box to resize (vboxmanage modifyhd). However, this supports only .vdi format and not .vmdk, hence you need to clone the .vmdk to .vdi file. use vboxmanage clonehd.

TIP: It is better to create 2 clones of .vmdk file and attached it to virtual box using GUI. 
why? because i could not control which HDD the vm booted from! always booted from hdd. And I think it is better / easier / safer to resize the non-root disk.

You may want to change format from .vdi to .vmdk. While i did it, I think it might be optional since vagrant packaging probably does it.

###Extend the partition
needs to be done inside projdir and inside vm (vagrant ssh)

	df -h
	identify the root disk
	sudo fdisk -l 
	to identify the disks
	
	I followed the steps at http://geekpeek.net/resize-filesystem-fdisk-resize2fs/. Here they are, in a nutshell:
	run inside vm (vagrant ssh)
	$ sudo fdisk /dev/sda
	> c
	> u
	> p
	> d
	> p
	> w
	$ sudo fdisk /dev/sda 
	> c
	> u
	> p
	> n
	> p
	> 1
	> (default)
	> (default)
	> p
	> w
	this extends it non-destructively.   
	exit  
	  
	vagrant halt  
	vagrant up  
	reboot the vm. check.  
	df -h
	
###rezise filesystem
	
	vagrant ssh
	sudo e2fsck -f /dev/sda  
	sudo resize2fs /dev/sda1
	exit
	
###remove unnecessary hdd from vm
Remove existing box-disk1.vmdk from virtual machine. Also remove the .vmdk (or .vdi) file that is was not resized. You may delete these.

TIP: SATA port 0 maps to /dev/sda, PORT 1 to /dev/sdb and so on.

###package box

Now package the box from **projdir**

I referred
> https://scotch.io/tutorials/how-to-create-a-vagrant-base-box-from-an-existing-one  
> http://linoxide.com/linux-how-to/setup-centos-7-vagrant-base-box-virtualbox/

and then ran following to reduce size of vm:

	remove temp files  
	sudo yum clean all  
	sudo rm -rf /tmp/*  
	sudo rm -f /var/log/wtmp /var/log/btmp  
	sudo dd if=/dev/zero of=/EMPTY bs=1M  
	sudo rm -f /EMPTY  
	cat /dev/null > ~/.bash_history && history -c && exit  

Now actual repackage to new box 

	vagrant package --output toohey-centos65.box  
	vagrant box add toohey-centos65 toohey-centos65.box  

Followed by:

	vagrant destroy  
	rm Vagrantfile  

You now have a toohey-centos65.box file that has been uploaded here. size 280 mb. 
