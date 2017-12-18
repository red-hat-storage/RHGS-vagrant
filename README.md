# RHGS 3.3.1 in Vagrant

A Vagrant setup for Red Hat Gluster Storage version 3.3 update 1.
This will setup as many RHGS nodes as you want with a number of bricks that you can define!  

## Requirements
* macOS with [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (starting 5.1.30) **or**
* RHEL 7.4/CentOS 1708/Fedora 27 with KVM/libvirt
* [Ansible](https://ansible.com) (starting 2.4.0.0)
* [Vagrant](https://www.vagrantup.com) (starting 1.9.1)
* git

## Installation instructions for Vagrant / Ansible

#### On RHEL 7.4

* make sure you are logged in as a user with `sudo` privileges
* make sure your system has the following repositories enabled (`yum repolist`)
  * rhel-7-server-rpms
  * rhel-7-server-extras-rpms
* install the requirements
  * `sudo yum groupinstall "Virtualization Host"`
  * `sudo yum install ansible git gcc libvirt-devel`
  * `sudo yum install https://releases.hashicorp.com/vagrant/2.0.1/vagrant_2.0.1_x86_64.rpm`
* start `libvirtd`
  * `sudo systemctl enable libvirtd`
  * `sudo systemctl start libvirtd`
* enable libvirt access for your current user
  * `sudo gpasswd -a $USER libvirt`
* as your normal user, install the libvirt plugin for vagrant
  * `vagrant plugin install vagrant-libvirt`

#### On Fedora 27

* make sure you are logged in as a user with `sudo` privileges
* make sure your system has the following repositories enabled (`dnf repolist`)
  * fedora
  * fedora-updates
* install the requirements
  * `sudo dnf install ansible git gcc libvirt-devel libvirt qemu-kvm`
  * `sudo dnf install vagrant vagrant-libvirt`
* start `libvirtd`
  * `sudo systemctl enable libvirtd`
  * `sudo systemctl start libvirtd`
* enable libvirt access for your current user
  * `sudo gpasswd -a $USER libvirt`
* as your normal user, install the libvirt plugin for vagrant
  * `vagrant plugin install vagrant-libvirt`

#### On macOS High Sierra

* install the requirements
  * install [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
  * install [Vagrant](https://www.vagrantup.com)
  * install [homebrew](https://brew.sh/)
  * install git
    * `brew install git`
  * install ansible
    * `pip install ansible`


## Get started
* You **must** be in the Red Hat VPN
* Clone this repository
  * `git clone https://github.com/dmesser/RHGS-vagrant.git`
* Goto the folder in which you cloned this repo
  * `cd RHGS-vagrant`
* Run `vagrant up`
  * Decide how many RHGS nodes and how many bricks you need
  * Decide if you want vagrant to initialize the cluster (`gdeploy`) for you
  * Wait a while

## Usage
* You can connect to each VM with `vagrant ssh` and the name of the VM you want to connect to
* Each VM is called RHGSx where x starts with 1
 * RHGS1 is your first VM and it counts up depending on the amount of VMs you spawn
* There are also other vagrant commands you should check out!
 * Try vagrant -h to find out about them
* *Always make sure you are in the git repo - vagrant only works in there!*
* modify the `RHGS_VERSION` parameter in the `Vagrantfile` for different combinations of OS and Gluster versions

## More info
* After starting the VMs, the hosts file is prepopulated and all packages are installed, a gdeploy.conf is in the home directory of the vagrant user
* If you decided to have vagrant initialize the cluster
  * gdeploy was executed with the gdeploy.conf file
  * cluster is peered
  * all block devices have been set up of VGs, LVs, formatted and mounted (`gdeploy`'s standard backend-setup)
  * brick directories have been created


### Creating your own vagrant box

If you - for whatever reason - do not want to use my prebuild box, you can create your own box very easy!  

**BEWARE** this is for advanced users only!

* Get [packer](https://www.packer.io/)
* Checkout the "packer" branch of this repository, follow the README

## Author
[Daniel Messer](mailto:dmesser@redhat.com) - [dmesser@redhat.com](mailto:dmesser@redhat.com) -
Technical Marketing Manager @ Red Hat

## Original Author
[Christopher Blum](https://github.com/zeichenanonym)
