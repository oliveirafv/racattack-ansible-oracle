## -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

#############################
## 20141111
## Alvaro Miranda
## http://kikitux.net
## alvaro at kikitux.net
#############################
##### BEGIN CUSTOMIZATION #####
#############################
#define number of nodes
num_APPLICATION 	= 0
num_LEAF_INSTANCES	= 0
num_DB_INSTANCES	= 2
#
#define number of cores for guest
num_CORE=1
#
#define memory for each type of node in MBytes
#
#for leaf nodes, the minimun can be  2300, otherwise pre-check will fail for
#automatic ulimit values calculated based on ram
#
#for database nodes, the minimum suggested is 3072
#
memory_APPLICATION 	= 1500
memory_LEAF_INSTANCES	= 2300
memory_DB_INSTANCES	= 3072
#        
#size of shared disk in GB
size_shared_disk	= 5
#location of shared disk
path_shared_disk = "."
#number of shared disks
count_shared_disk = 4
#
#############################
##### END CUSTOMIZATION #####
#############################

inventory_ansible = File.open("stagefiles/ansible-oracle/inventory/racattack","w")
inventory_ansible << "[application]\n"
(1..num_APPLICATION).each do |i|
  inventory_ansible << "collaba#{i}\n"
end
inventory_ansible << "[leaf]\n"
(1..num_LEAF_INSTANCES).each do |i|
  inventory_ansible << "collabl#{i}\n"
end
inventory_ansible << "[hub]\n"
(1..num_DB_INSTANCES).each do |i|
  inventory_ansible << "collabn#{i}.racattack ansible_ssh_user=root ansible_ssh_pass=root\n"
end
inventory_ansible << "[racattack:children]\n"
inventory_ansible << "leaf\n"	if num_LEAF_INSTANCES > 0
inventory_ansible << "hub\n"	if num_DB_INSTANCES > 0
inventory_ansible.close

$etc_hosts_script = <<SCRIPT
#!/bin/bash
grep PEERDNS /etc/sysconfig/network-scripts/ifcfg-eth0 || echo 'PEERDNS=no' >> /etc/sysconfig/network-scripts/ifcfg-eth0
echo "overwriting /etc/resolv.conf"
cat > /etc/resolv.conf <<EOF
nameserver 192.168.78.51
nameserver 192.168.78.52
search racattack
EOF

cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost6 localhost6.localdomain6
EOF

SCRIPT

#variable used to provide information only once
give_info ||=true

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  #config.vm.box = "racattack/oracle65"
  config.vm.box = "kikitux/oracle6-racattack"

  ## Virtualbox modifications
  ## we first setup memory and cpu
  ## we create shared disks if they don't exists
  ## we later attach the disk to the vms
  ## we attach to each vm, as in the future we may want to have say 2db + 2app cluster
  ## we can attach 2 shared disk for db to the db nodes only
  ## and 2 other shared disks for the app

  if File.directory?("stagefiles")
    # our shared folder for scripts
    config.vm.synced_folder "stagefiles", "/media/stagefiles", :mount_options => ["dmode=555","fmode=444","gid=54321"]

    #run some scripts
    config.vm.provision :shell, :inline => $etc_hosts_script
  end

  if File.directory?("12cR1")
    # our shared folder for oracle 12c installation files
    config.vm.synced_folder "12cR1", "/media/sf_12cR1", :mount_options => ["dmode=775","fmode=775","uid=54320","gid=54321"]
  end


  ## IMPORTANT
  ## vagrant work up to down, high node goes first
  ## so when node 1 is ready, we can configure rac and all nodes will be up

  (1..num_APPLICATION).each do |i|
    # this is to start machines higher to lower
    i = num_APPLICATION+1-i
    config.vm.define vm_name = "collaba%01d" % i do |config|
      puts " "
      config.vm.hostname = "#{vm_name}.racattack"
      lanip = "192.168.78.#{i+90}"
      puts vm_name + " eth1 lanip  :" + lanip
      config.vm.provider :virtualbox do |vb|
        vb.name = vm_name + "." + Time.now.strftime("%y%m%d%H%M")
        vb.customize ["modifyvm", :id, "--memory", memory_APPLICATION]
        vb.customize ["modifyvm", :id, "--cpus", num_CORE]
        vb.customize ["modifyvm", :id, "--groups", "/collab"]
      end
      config.vm.network :private_network, ip: lanip
    end
  end

  (1..num_LEAF_INSTANCES).each do |i|
    # this is to start machines higher to lower
    i = num_LEAF_INSTANCES+1-i
    config.vm.define vm_name = "collabl%01d" % i do |config|
      puts " "
      config.vm.hostname = "#{vm_name}.racattack"
      lanip = "192.168.78.#{i+70}"
      puts vm_name + " eth1 lanip  :" + lanip
      privip = "172.16.100.#{i+70}"
      puts vm_name + " eth2 privip :" + privip
      config.vm.provider :virtualbox do |vb|
        vb.name = vm_name + "." + Time.now.strftime("%y%m%d%H%M")
        vb.customize ["modifyvm", :id, "--memory", memory_LEAF_INSTANCES]
        vb.customize ["modifyvm", :id, "--cpus", num_CORE]
        vb.customize ["modifyvm", :id, "--groups", "/collab"]
      end
      config.vm.network :private_network, ip: lanip
      config.vm.network :private_network, ip: privip
    end
  end

  (1..num_DB_INSTANCES).each do |i|
    # this is to start machines higher to lower
    i = num_DB_INSTANCES+1-i
    config.vm.define vm_name = "collabn%01d" % i do |config|
      puts " "
      config.vm.hostname = "#{vm_name}.racattack"
      lanip = "192.168.78.#{i+50}"
      puts vm_name + " eth1 lanip  :" + lanip
      privip = "172.16.100.#{i+50}"
      puts vm_name + " eth2 privip :" + privip
      config.vm.provider :virtualbox do |vb|
        vb.name = vm_name + "." + Time.now.strftime("%y%m%d%H%M")
        vb.customize ["modifyvm", :id, "--memory", memory_DB_INSTANCES]
        vb.customize ["modifyvm", :id, "--cpus", num_CORE]
        vb.customize ["modifyvm", :id, "--groups", "/collab"]
        #first shared disk port
        port=2
        #how many shared disk
        (1..count_shared_disk).each do |disk|
          file_to_dbdisk = "#{path_shared_disk}/racattack-shared-disk"
          if !File.exist?("#{file_to_dbdisk}#{disk}.vdi")
            unless give_info==false
              puts "on first boot shared disks will be created, this will take some time"
              give_info=false
            end
            vb.customize ['createhd', '--filename', "#{file_to_dbdisk}#{disk}.vdi", '--size', (size_shared_disk * 1024).floor, '--variant', 'fixed']
            vb.customize ['modifyhd', "#{file_to_dbdisk}#{disk}.vdi", '--type', 'shareable']
          end
          vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', port, '--device', 0, '--type', 'hdd', '--medium', "#{file_to_dbdisk}#{disk}.vdi"]
          port=port+1
        end
      end
      config.vm.network :private_network, ip: lanip
      config.vm.network :private_network, ip: privip
      #config.vm.provision :shell, :inline => "sh /media/stagefiles/setup_ssh_known_hosts.sh"
      if vm_name == "collabn1" 
        puts vm_name + " dns server role is master"
        config.vm.provision :shell, :inline => "sh /media/stagefiles/named_master.sh"
        if ENV['setup'] 
          config.vm.provision :shell, :inline => "sh /media/stagefiles/run_ansible_playbook.sh"
        end
      end
      if vm_name == "collabn2" 
        puts vm_name + " dns server role is slave"
        config.vm.provision :shell, :inline => "sh /media/stagefiles/named_slave.sh"
      end
    end
  end


  # This network is optional, that's why is at the end

  # Create a public network, which generally matched to bridged network.
  #default will ask what network to bridge
  #config.vm.network :public_network

  # OSX
  # 1) en1: Wi-Fi (AirPort)
  # 2) en0: Ethernet

  # Windows

  # Linux laptop
  # 1) wlan0
  # 2) eth0
  # 3) lxcbr0

  # Linux Desktop
  # 1) eth0
  # 2) eth1
  # 3) lxcbr0
  # 4) br0

  # on OSX to the wifi
  #config.vm.network :public_network, :bridge => 'en1: Wi-Fi (AirPort)'

end
