# Packer.io-based image creation

The images for the Vagrant setup are created via [packer](https://www.packer.io). Follow these instructions to create your own images.

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
* run packer
  * `packer build packer-virtualbox-rhel-7.json`

## Author
[Daniel Messer](mailto:dmesser@redhat.com)
