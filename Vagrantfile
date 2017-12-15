# -*- mode: ruby -*-
# vi: set ft=ruby :

# Some variables we need below
VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))

#Disable parallel runs - breaks peer probe in the end
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

#################
# Set RHGS version
RHGS_VERSION = "rhgs-3.3.1-rhel-7"

# Currently available versions:
# rhgs-3.3.1-rhel-7
# rhgs-3.3.1-centos-7
#################

#################
# General VM settings applied to all VMs
#################
VMCPU = 1
VMMEM = 1024
#################

numberOfVMs = 0
numberOfDisks = -1
clusterInit = -1

if ARGV[0] == "up"

  print "\n\e[1;37mHow many RHGS nodes do you want me to provision for you? [2] \e[32m"
  while numberOfVMs < 2
    numberOfVMs = $stdin.gets.strip.to_i
    if numberOfVMs == 0 # The user pressed enter without input or we cannot parse the input to a number
      numberOfVMs = 2
    elsif numberOfVMs < 2
      print "\e[31mWe need at least 2 VMs ;) Try again \e[32m"
    end
  end

  print "\e[1;37mHow many disks do you need per VM for bricks? [2] \e[32m"

  while numberOfDisks < 1
    numberOfDisks = $stdin.gets.strip.to_i
    if numberOfDisks == 0 # The user pressed enter without input or we cannot parse the input to a number
      numberOfDisks = 2
    elsif numberOfDisks < 1
      print "\e[31mWe need at least 1 disk ;) Try again \e[32m"
    end
  end

  print "\e[1;37mDo you want me to initialize the cluster for you? [no] \e[32m"

  while clusterInit == -1
    response = $stdin.gets.strip.to_s

    if response == 'yes' or response == 'y'
      clusterInit = 1
    elsif response == 'no' or response == 'n' or response == ''
      clusterInit = 0
    else
      print "\e[31mPlease enter 'yes' or 'no'\e[32m"
    end
  end


  environment = open('vagrant_env.conf', 'w')
  environment.puts("# BEWARE: Do NOT modify ANY settings in here or your vagrant environment will be messed up")
  environment.puts(numberOfVMs.to_s)
  environment.puts(numberOfDisks.to_s)
  environment.close

  print "\e[32m\nOK I will provision #{numberOfVMs} VMs for you and each one will have #{numberOfDisks} disks for bricks\e[37m\n"

  if clusterInit == 1
    print "\e[32m\nAlso, I will initialize the cluster for you\e[37m\n\n"
  else
    print "\e[32m\nAlso, I will not initialize the cluster but leave a gdeploy.conf for your convenience\e[37m\n\n"
  end

  system "sleep 1"
else # So that we destroy and can connect to all VMs...
  begin
    environment = open('vagrant_env.conf', 'r')
    environment.readline # Skip the comment on top
    numberOfVMs = environment.readline.to_i
    numberOfDisks = environment.readline.to_i
    environment.close
  rescue # File was deleted or is unreadable and we just don't care...
    numberOfVMs = 2
    numberOfDisks = 2
    print "\e[31m\nWARNING! Couldn't find file vagrant_env.conf! I will assume you have provisioned #{numberOfVMs} VMs\e[37m\n\n"
  end



  if ARGV[0] != "ssh-config"
    puts "Detected settings from previous vagrant up:"
    puts "  We deployed #{numberOfVMs} VMs with each #{numberOfDisks} disks"
    puts ""
  end
end

def vBoxAttachDisks(numDisk, provider, boxName)
  for i in 1..numDisk.to_i
    file_to_disk = File.join(VAGRANT_ROOT, 'disks', ( boxName + '-' +'disk' + i.to_s + '.vdi' ))
    unless File.exist?(file_to_disk)
      provider.customize ['createhd', '--filename', file_to_disk, '--size', 100 * 1024] # 30GB brick device
    end
    provider.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', i, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  end
end

def libvirtAttachDisks(numDisk, provider)
  for i in 1..numDisk.to_i
    provider.storage :file, :bus => 'virtio', :size => '100G'
  end
end

# Vagrant config section starts here
Vagrant.configure(2) do |config|

  config.vm.provider "virtualbox" do |vb, override|
    override.vm.box_url = "http://file.rdu.redhat.com/~dmesser/rhgs-vagrant/virtualbox-#{RHGS_VERSION}.box"
  end

  config.vm.provider "libvirt" do |libvirt, override|
    override.vm.box_url = "file:///tmp/rhgs-rhel-7-virtualbox/libvirt-#{RHGS_VERSION}.box"
    libvirt.storage_pool_name = ENV['LIBVIRT_STORAGE_POOL'] || 'default'
  end

  (1..numberOfVMs).each do |vmNum|
    config.vm.define "RHGS#{vmNum.to_s}" do |machine|

      # Provider-indepent options
      machine.vm.hostname = "RHGS#{vmNum.to_s}"

      machine.vm.provider "virtualbox" do |vb, override|

        # This will be the private VM-only network where GlusterFS traffic will flow
        override.vm.network "private_network", type: "dhcp", nic_type: "virtio", auto_config: false
        override.vm.box = RHGS_VERSION

        # Make this a linked clone
        vb.linked_clone = true

        # Set VM resources
        vb.memory = VMMEM
        vb.cpus = VMCPU

        # Don't display the VirtualBox GUI when booting the machine
        vb.gui = false
        vb.name = "RHGS#{vmNum.to_s}"

        vBoxAttachDisks( numberOfDisks, vb, "RHGS#{vmNum.to_s}" )

        # Accelerate SSH / Ansible connections (https://github.com/mitchellh/vagrant/issues/1807)
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      end

      machine.vm.provider "libvirt" do |libvirt, override|
        override.vm.box = RHGS_VERSION
        override.vm.network "private_network", type: "dhcp", auto_config: false
        override.vm.synced_folder ".", "/vagrant", disabled: true

        libvirt.memory = VMMEM
        libvirt.cpus = VMCPU
        libvirt.nic_model_type = "virtio"
        libvirt.disk_bus = "virtio"
        libvirt.username = "root"
        # libvirt.graphics_type = "spice"
        # libvirt.video_type = "qxl"

        libvirtAttachDisks( numberOfDisks, libvirt )
      end

      if vmNum == numberOfVMs
        machine.vm.provider "virtualbox" do |virtualbox,override|
          override.vm.provision :ansible do |ansible|
            ansible.limit = "all"
            ansible.extra_vars = { provider: "virtualbox" }
            ansible.playbook = "ansible/prepare-environment.yml"
          end

          if clusterInit == 1
            override.vm.provision "shell", privileged: false, inline: "gdeploy -c gdeploy.conf"
          end
        end

        machine.vm.provider "libvirt" do |libvirt,override|
          override.vm.provision :ansible do |ansible|
            ansible.limit = "all"
            ansible.extra_vars = { provider: "libvirt" }
            ansible.playbook = "ansible/prepare-environment.yml"
          end

          # awkward reptition due to lack of provisioner priority in Vagrant
          if clusterInit == 1
            override.vm.provision "shell", privileged: false, inline: "gdeploy -c gdeploy.conf"
          end
        end
      end
    end
  end

end
