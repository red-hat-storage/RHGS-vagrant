# -*- mode: ruby -*-
# vi: set ft=ruby :

# Some variables we need below
VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))

#Disable parallel runs - breaks peer probe in the end
ENV['VAGRANT_NO_PARALLEL'] = 'yes'

#################
# Set RHGS version
RHGS_VERSION = "RHGS3.2.0"

# Currently available versions:
# RHGS3.0.4
# RHGS3.1.1
# RHGS3.1.2
# RHGS3.1.3
# RHGS3.2.0
# RHS2.1.0
#################

#################
# General VM settings applied to all VMs
#################
VMCPU = 3
VMMEM = 1536
#################


numberOfVMs = 0
numberOfDisks = -1

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

  environment = open('vagrant_env.conf', 'w')
  environment.puts("# BEWARE: Do NOT modify ANY settings in here or your vagrant environment will be messed up")
  environment.puts(numberOfVMs.to_s)
  environment.puts(numberOfDisks.to_s)
  environment.close

  print "\e[32m\nOK I will provision #{numberOfVMs} VMs for you and each one will have #{numberOfDisks} disks for bricks\e[37m\n\n"
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


hostsFile = "192.168.10.200 RHGS1\n"
(2..numberOfVMs).each do |num|
  hostsFile += "192.168.10.#{( 98 + num).to_s} RHGS#{num.to_s}\n"
end


def vBoxAttachDisks(numDisk, provider, boxName)
  for i in 1..numDisk.to_i
    file_to_disk = File.join(VAGRANT_ROOT, 'disks', ( boxName + '-' +'disk' + i.to_s + '.vdi' ))
    unless File.exist?(file_to_disk)
      provider.customize ['createhd', '--filename', file_to_disk, '--size', 100 * 1024] # 30GB brick device
    end
    provider.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', i, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  end
end

def lvAttachDisks(numDisk, provider)
  for i in 1..numDisk.to_i
    provider.storage :file, :size => '100G'
  end
end

# Vagrant config section starts here
Vagrant.configure(2) do |config|
  config.vm.box_url = "http://file.rdu.redhat.com/~cblum/vagrant-storage/#{RHGS_VERSION}.json"
  (2..numberOfVMs).each do |vmNum|
    config.vm.define "RHGS#{vmNum.to_s}" do |copycat|
      # This will be the private VM-only network where GlusterFS traffic will flow
      copycat.vm.network "private_network", ip: ( "192.168.10." + (98 + vmNum).to_s )
      copycat.vm.hostname = "RHGS#{vmNum.to_s}"

      copycat.vm.provider "virtualbox" do |vb, override|
        override.vm.box = RHGS_VERSION
#        override.vm.synced_folder '.', '/vagrant', type: 'rsync'

        # Don't display the VirtualBox GUI when booting the machine
        vb.gui = false
        vb.name = "RHGS#{vmNum.to_s}-RHEL7"
      
        # Customize the amount of memory and vCPU in the VM:
        vb.memory = VMMEM
        vb.cpus = VMCPU

        vBoxAttachDisks( numberOfDisks, vb, "RHGS#{vmNum.to_s}" )
      end

      copycat.vm.provider "libvirt" do |lv, override|
        override.vm.box = RHGS_VERSION
        override.vm.synced_folder '.', '/vagrant', type: 'rsync'
      
        # Customize the amount of memory and vCPU in the VM:
        lv.memory = VMMEM
        lv.cpus = VMCPU

        lvAttachDisks( numberOfDisks, lv )
      end
      copycat.vm.post_up_message = "\e[37mBuilding of this VM is finished \nYou can access it now with: \nvagrant ssh RHGS#{vmNum.to_s}\e[32m"

    end
  end
  

  config.vm.define "RHGS1" do |mainbox|
    # This will be the private VM-only network where GlusterFS traffic will flow
    mainbox.vm.network "private_network", ip: '192.168.10.200'
    mainbox.vm.hostname = 'RHGS1'
    
    mainbox.vm.provider "virtualbox" do |vb, override|
      override.vm.box = RHGS_VERSION
#      override.vm.synced_folder '.', '/vagrant', type: 'rsync'

      # Don't display the VirtualBox GUI when booting the machine
      vb.gui = false
      vb.name = "RHGS1-RHEL7"
    
      # Customize the amount of memory and vCPU in the VM:
      vb.memory = VMMEM
      vb.cpus = VMCPU

      vBoxAttachDisks( numberOfDisks, vb, 'RHGS1' )
    end
    
    mainbox.vm.provider "libvirt" do |lv, override|
      override.vm.box = RHGS_VERSION
      override.vm.synced_folder '.', '/vagrant', type: 'rsync'
    
      # Customize the amount of memory and vCPU in the VM:
      lv.memory = VMMEM
      lv.cpus = VMCPU

      lvAttachDisks( numberOfDisks, lv )
    end

    command = ''
    (2..numberOfVMs).each do |num|
      command += "sudo gluster peer probe rhgs#{num.to_s};"
    end

    mainbox.vm.provision "shell",
      inline: command
    
    csshCmd = "vagrant ssh-config > ssh_conf; csshx --ssh_args '-F #{VAGRANT_ROOT}/ssh_conf' RHGS1 "
    (2..numberOfVMs).each do |num|
      csshCmd += "RHGS#{num.to_s} "
    end

    mainbox.vm.post_up_message = "If you don't see any text below, it's because the text color is white ;)\n\e[37mBuilding of this VM is finished \nYou can access it now with: \nvagrant ssh RHGS1\nI already connected the RHGS nodes with gluster peer probe for your convenience\n\n csshX Command line:\n#{csshCmd}\e[32m"

  end


  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.

  # The flow is outside->in so that this will run before all node specific Shell skripts mentioned above!

  config.vm.provision "shell", inline: <<-SHELL
    sudo sed --in-place --expression="s/UUID=.*$/UUID=$(cat /proc/sys/kernel/random/uuid)/" /var/lib/glusterd/glusterd.info
    sudo /bin/systemctl restart  glusterd.service
    sudo iptables -F
    echo '#{hostsFile}' | sudo tee -a /etc/hosts
    # Fix localhost for https://github.com/red-hat-storage/RHGS-vagrant/issues/2
    sed -i 's/127\.0\.0\.1.*/127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4/g' /etc/hosts
    yes | sudo /usr/lib/glusterfs/.unsupported/rhs-system-init.sh; true
  SHELL

  config.push.define "local-exec" do |push|

    push.inline = <<-SCRIPT
      rsync -avr --exclude 'disk*' --exclude '.vagrant' --exclude '.DS_Store' --exclude 'ssh_conf' --exclude 'vagrant_env.conf' . storchris.dorf.rwth-aachen.de:/var/www/vagrant/RHGS-RHEL7/
    SCRIPT
  end


end
