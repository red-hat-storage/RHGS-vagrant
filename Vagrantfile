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

  print "\n\e[1;37mHow many RHGS nodes do you want me to provision for you? Default: 2 \e[32m"
  while numberOfVMs < 2 or numberOfVMs > 99
    numberOfVMs = $stdin.gets.strip.to_i
    if numberOfVMs == 0 # The user pressed enter without input or we cannot parse the input to a number
      numberOfVMs = 2
    elsif numberOfVMs < 2
      print "\e[31mWe need at least 2 VMs ;) Try again \e[32m"
    elsif numberOfVMs > 99
      print "\e[31mWe don't support more than 99 VMs - Try again \e[32m"
    end
  end

  print "\e[1;37mHow many disks do you need per VM for bricks? Default: 2 \e[32m"

  while numberOfDisks < 1
    numberOfDisks = $stdin.gets.strip.to_i
    if numberOfDisks == 0 # The user pressed enter without input or we cannot parse the input to a number
      numberOfDisks = 2
    elsif numberOfDisks < 1
      print "\e[31mWe need at least 1 disk ;) Try again \e[32m"
    end
  end

  print "\e[1;37mDo you want me to initialize the cluster for you? Default: no \e[32m"

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
    print "\e[32m\nI will initialize the cluster for you\e[37m\n\n"
  else
    print "\e[32m\nI will not initialize the cluster but leave a gdeploy.conf for your convenience\e[37m\n\n"
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

# Vagrant config section starts here
Vagrant.configure(2) do |config|
  config.vm.box_url = "http://file.rdu.redhat.com/~dmesser/rhgs-vagrant/virtualbox-#{RHGS_VERSION}.box"

  (1..numberOfVMs).each do |vmNum|
    config.vm.define "RHGS#{vmNum.to_s}" do |machine|
      # This will be the private VM-only network where GlusterFS traffic will flow
      machine.vm.network "private_network", type: "dhcp", nic_type: "virtio", auto_config: false
      machine.vm.hostname = "RHGS#{vmNum.to_s}"

      machine.vm.provider "virtualbox" do |vb, override|
        override.vm.box = RHGS_VERSION

        # Make this a linked clone
        vb.linked_clone = true

        # Don't display the VirtualBox GUI when booting the machine
        vb.gui = false
        vb.name = "RHGS#{vmNum.to_s}-RHEL7"

        # Customize the amount of memory and vCPU in the VM:
        vb.memory = VMMEM
        vb.cpus = VMCPU

        vBoxAttachDisks( numberOfDisks, vb, "RHGS#{vmNum.to_s}" )

        # Accelerate SSH / Ansible connections (https://github.com/mitchellh/vagrant/issues/1807)
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      end

      if vmNum == numberOfVMs
        machine.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "ansible/prepare-environment.yml"
        end

        machine.vm.provision :ansible do |ansible|
          ansible.limit = "all"

          config.vm.provider "virtualbox" do |vb|
            ansible.extra_vars = { provider: "virtualbox" }
          end

          ansible.playbook = "ansible/prepare-gdeploy.yml"
        end

        if clusterInit == 1
          machine.vm.provision "shell", privileged: false, inline: "gdeploy -c gdeploy.conf"
        end
      end
    end
  end
end
