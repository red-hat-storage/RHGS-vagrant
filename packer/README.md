# Packer.io-based image creation

The images for the Vagrant setup are created via [packer](https://www.packer.io). Follow these instructions to create your own images.

## Contents

This directory holds packer configuration files for the following combinations of storage software, operating systems and virtual machine platforms:

|Storage Software|Operating System|VM Platform|Packer file|
|---|---|---|---|
|Red Hat Gluster Storage 3.3.0|Red Hat Enterprise Linux 7|VirtualBox|packer-virtualbox-rhgs-3.3.0-rhel-7.json|


## Requirements
* [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (starting 5.1.30)
* [Packer](https://www.packer.io) (starting 1.1.3)
* [Ansible](https://ansible.com) (starting 2.4.2.0)
* git
* RHEL 7.4 ISO

## Get started
* Determine a non-existing (!) output directory for the resulting OVA (e.g. /tmp/rhgs-rhel-7-virtualbox)
* Download RHEL 7.4
  * put it somewhere (e.g. /tmp/rhel-server-7.4-x86_64-dvd.iso)
* Clone this repository
  * `git clone -b packer https://github.com/dmesser/RHGS-vagrant.git`
* Goto the folder in which you cloned this repo
  * `cd RHGS-vagrant/packer`
* Tell packer where to find your ISO, e.g.
  * `export ISO_FILE_PATH=file:///tmp/rhel-server-7.4-x86_64-dvd.iso`
* Tell packer where to store the resulting OVA
  * `export OUTPUT_DIRECTORY=/tmp/rhgs-rhel-7-virtualbox`
* run packer, e.g.
  * `packer build packer-virtualbox-rhgs-3.3.0-rhel-7`

## Author
[Daniel Messer](mailto:dmesser@redhat.com)
