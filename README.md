# RHGS 3.3.1 in Vagrant

A Vagrant setup for Red Hat Gluster Storage version 3.3 update 1.
This will setup as many RHGS nodes as you want with a number of bricks that you can define!
Optionally you can choose to deploy the management UI [tendrl](github.com/tendrl).

## Requirements
* macOS with [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (starting 5.1.30) **or**
* RHEL 7.4/Fedora 27 with KVM/libvirt
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
    * `brew install ansible`

## Make sure you are up-to-date

If you have satisfied all the requirements and you ran RHGS-vagrant before, ensure from time to time that you are using the most current images:

* run `git pull` in the RHGS-vagrant directory to pull the latest updates for the Vagrant automation
* from time-to-time new images are released (especially for async version updates)
  * run `rm ~/.vagrant.d/boxes/rhgs-*.box` and `rm ~/.vagrant.d/boxes/tendrl-*.box` to delete older Vagrant images
  * on VirtualBox - remove the VM instances named `packer-tendrl-server-...` and `packer-rhgs-node-...` (these are base images for the clones)
  * on libvirt
    * run `virsh vol-list default` to list all images in your `default` storage pool (adjust the name if you are using a different one)
    * run `virsh vol-delete rhgs-node-... default` and  `virsh vol-delete tendrl-server-... default` to delete the images starting with `rhgs-node-...` and `tendrl-server-...` (replace with full name) from the default pool

Next time you do `vagrant up` it will automatically pull new images.

## Get started
* You **must** be in the Red Hat VPN
* Clone this repository
  * `git clone https://github.com/red-hat-storage/RHGS-vagrant.git`
* Goto the folder in which you cloned this repo
  * `cd RHGS-vagrant`
  * if you are on RHEL/Fedora and your don't want your libvirt storage domain `default` to be used, override the storage domain like this
    * `export LIBVIRT_STORAGE_POOL=images`
* Run `vagrant up`
  * Decide how many RHGS nodes and how many bricks you need
  * Decide if you want vagrant to initialize the cluster (`gdeploy`) for you
  * If you opted to initialize the cluster, decide whether you want to deploy tendrl
  * Wait a while

## Usage
* *Always make sure you are in the git repo - vagrant only works in there!*
* After `vagrant up` you can connect to each VM with `vagrant ssh` and the name of the VM you want to connect to
* Each VM is called `RHGSx` where x starts with 1
  * RHGS1 is your first VM and it counts up depending on the amount of VMs you spawn
  * There is an additional VM called `TENDRL` which hosts the Gluster Web Admin Server if you selected to deploy it (URL is displayed at the end of `vagrant up`)
* There are also other vagrant commands you should check out!
  * if you want to throw away everything: `vagrant destroy -f`
  * if you want to freeze the VMs and continue later: `vagrant suspend`
  * Try `vagrant -h` to find out about them
  * if you run `vagrant up` again you without running `vagrant destroy` before you will overwrite your configuration and vagrant may loose track of some VMs (it's safe to remove them manually)
* modify the `RHGS_VERSION` / `TENDRL_VERSION` parameter in the `Vagrantfile` for different combinations of OS and Gluster/Tendrl versions
* modify the `VMMEM` and `VMCPU` variables in the Vagrant file to change RHGS VM resources, adjust `VMDISK` to change brick device sizes

## What happens under the covers
* After starting the RHGS VMs:
  * the hosts file is prepopulated
  * all glusters packages are pre-installed (allows you to continue offline)
  * the RHEL images are subscribed to YUM repositories on the RHT VPN
  * a gdeploy.conf is in the home directory of the vagrant user
* If you decided to have vagrant initialize the cluster
  * gdeploy was executed with the gdeploy.conf file
  * cluster is peered
  * all block devices have been set up of VGs, LVs, formatted and mounted (`gdeploy`'s standard backend-setup)
  * brick directories have been created
* If you decided to deploy tendrl
  * an additional VM will run tendrl server components
  * the Ansible inventory and tendrl install playbook have been generated
  * the installation playbook has been executed on the Tendrl server and RHGS nodes
  * the Tendrl UI is reachable on the IP address of the eth1 adapter of the Tendrl VM (the URL also displayed after the installer finished)


### Creating your own vagrant box

If you - for whatever reason - do not want to use my prebuilt box, you can create your own box very easy!  

**BEWARE** this is for advanced users only!

* Get [packer](https://www.packer.io/)
* `git checkout` the "packer" branch of this repository, follow the README

## Author
[Daniel Messer](mailto:dmesser@redhat.com) - [dmesser@redhat.com](mailto:dmesser@redhat.com) -
Technical Marketing Manager @ Red Hat

## Original Author
[Christopher Blum](https://github.com/zeichenanonym)
