# RHGS 3.1.2 in Vagrant

A Vagrant setup for Red Hat Gluster Storage version 3.1 update 2.  
This will setup as many RHGS nodes as you want with a number of bricks that you can define!  

## Requirements
* [Virtualbox](https://www.virtualbox.org/wiki/Downloads) or KVM
* [Vagrant](https://www.vagrantup.com/) (latest version)
* Git

## Get started
* Clone this repository
 * `git clone https://github.com/red-hat-storage/RHGS-vagrant.git`
* Goto the folder in which you cloned this repo
 * `cd RHGS-vagrant`
* Run `vagrant up`
* Decide how many RHGS nodes and how many bricks you need
* Wait a while

## Usage
* You can connect to each VM with `vagrant ssh` and the name of the VM you want to connect to
* Each VM is called RHGSx where x starts with 1
 * RHGS1 is your first VM and it counts up depending on the amount of VMs you spawn
* There are also other vagrant commands you should check out!
 * Try vagrant -h to find out about them
* *Always make sure you are in the git repo - vagrant only works in there!*

## More info
* After starting the VMs, the hosts file is prepopulated and the VMs are peered
* I love to use [csshx](https://www.outsideopen.com/csshx/) to control multiple VMs at once
 * After creating the VMs, you will get a one-line command for csshx to control all your created VMs at once
* For quick and dirty brick generation, check out `/usr/lib/glusterfs/.unsupported/rhs-system-init.sh`

### Creating your own vagrant box

If you - for whatever reason - do not want to use my prebuild box, you can create your own box very easy!  
  
**BEWARE** this is for advanced users only!

* Get [packer](https://www.packer.io/)
* Look at the packer.json file and modify at least the iso location for the RHGS installation
* Run `packer build --only=virtualbox-iso -var 'RHN_user=[YOUR RHT USER]' -var 'RHN_password=[YOUR RHT PASSWORD]' packer.json`
 * This will create a virtualbox vagrant box and put it in the current directory
* `vagrant box add` this new box
* Modify the Vagrantfile to use your box
* Have fun ;)

## Author
[Christopher Blum](mailto:cblum@redhat.com) - [cblum@redhat.com](mailto:cblum@redhat.com)
Associate Storage Consultant @ Red Hat
