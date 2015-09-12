# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'

Vagrant.require_version ">= 1.6.0"

CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), "user-data")
CONFIG = File.join(File.dirname(__FILE__), "config.rb")

# Defaults for config options defined in CONFIG
$num_instances = 1
$instance_name_prefix = "core"
$update_channel = "alpha"
$image_version = "current"
$enable_serial_logging = false
$share_home = true
$vm_gui = false
$vm_memory = 4096
$vm_cpus = 2
$shared_folders = {'./gwydyon' => '/home/core/share/gwydyon'}
$forwarded_ports = {}
$expose_docker_tcp = 14632

# Attempt to apply the deprecated environment variable NUM_INSTANCES to
# $num_instances while allowing config.rb.bak to override it
if ENV["NUM_INSTANCES"].to_i > 0 && ENV["NUM_INSTANCES"]
  $num_instances = ENV["NUM_INSTANCES"].to_i
end

if File.exist?(CONFIG)
  require CONFIG
end

# Use old vb_xxx config variables when set
def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-%s" % $update_channel
  if $image_version != "current"
    config.vm.box_version = $image_version
  end
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/%s/coreos_production_vagrant.json" % [$update_channel, $image_version]

  ["vmware_fusion", "vmware_workstation"].each do |vmware|
    config.vm.provider vmware do |v, override|
      override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/%s/coreos_production_vagrant_vmware_fusion.json" % [$update_channel, $image_version]
    end
  end

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%02d" % [$instance_name_prefix, i] do |config|
      config.vm.hostname = vm_name

      if $enable_serial_logging
        logdir = File.join(File.dirname(__FILE__), "log")
        FileUtils.mkdir_p(logdir)

        serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
        FileUtils.touch(serialFile)

        ["vmware_fusion", "vmware_workstation"].each do |vmware|
          config.vm.provider vmware do |v, override|
            v.vmx["serial0.present"] = "TRUE"
            v.vmx["serial0.fileType"] = "file"
            v.vmx["serial0.fileName"] = serialFile
            v.vmx["serial0.tryNoRxLoss"] = "FALSE"
          end
        end

        config.vm.provider :virtualbox do |vb, override|
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      if $expose_docker_tcp
         config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
      end

      # $forwarded_ports.each do |guest, host|
      #   config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
      # end

      ["vmware_fusion", "vmware_workstation"].each do |vmware|
        config.vm.provider vmware do |v|
          v.gui = vm_gui
          v.vmx['memsize'] = vm_memory
          v.vmx['numvcpus'] = vm_cpus
        end
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = vm_gui
        vb.memory = vm_memory
        vb.cpus = vm_cpus
      end

      #ip = "172.17.8.#{i+100}"
      #config.vm.network :private_network, ip: ip

      Vagrant.configure("2") do |config|
        config.vm.network "public_network"
      end
      # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
      #config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']
      $shared_folders.each_with_index do |(host_folder, guest_folder), index|
        config.vm.synced_folder host_folder.to_s, guest_folder.to_s, id: "core-share%02d" % index, nfs: true, mount_options: ['nolock,vers=3']
      end

      if $share_home
        config.vm.synced_folder ENV['HOME'], ENV['HOME'], id: "home", :nfs => true, :mount_options => ['nolock,vers=3']
      end

      if File.exist?(CLOUD_CONFIG_PATH)
        config.vm.provision :file, :source => "#{CLOUD_CONFIG_PATH}", :destination => "/tmp/vagrantfile-user-data"
        config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end

      ### Solr specific stuff
      # config.vm.network "private_network", ip: "172.17.8.150"
      # config.vm.synced_folder ".", "/home/core/share", id: "core-solr", :nfs => true,  :mount_options   => ['nolock,vers=4']
      config.vm.network "forwarded_port", guest: 8983, host: 8983, auto_correct: true
      config.vm.network "forwarded_port", guest: 8984, host: 8984, auto_correct: true
      config.vm.network "forwarded_port", guest: 8985, host: 8985, auto_correct: true
      config.vm.network "forwarded_port", guest: 2181, host: 2181, auto_correct: true
      config.vm.network "forwarded_port", guest: 2888, host: 2888, auto_correct: true
      config.vm.network "forwarded_port", guest: 3888, host: 3888, auto_correct: true
      # config.vm.network "forwarded_port", guest: 8790, host: 8790, auto_correct: false

      # config.vm.provision :shell, :inline => 'docker run --name zookeeper -d -p 2181:2181 -p 2888:2888 -p 3888:3888 jplock/zookeeper', :privileged => true
      # config.vm.provision :shell, :inline => "docker run --name solr1 --link zookeeper:ZK -d -p 8983:8983 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
      # config.vm.provision :shell, :inline => "docker run --name solr2 --link zookeeper:ZK -d -p 8984:8983 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
       # config.vm.provision :shell, :inline => "docker run --name solr3 --link zookeeper:ZK -d -p 8985:8983 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
      # config.vm.provision :shell, :inline => 'docker exec -i -t solr1 /opt/solr/bin/solr create_collection -c collection1 -shards 3 -p 8983', :privileged => true


      # &&
      # chmod +x /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose && docker-compose --version

      config.vm.provision :shell, :inline => "cd /home/core/share/gwydyon/ && mkdir dc && curl -L https://github.com/docker/compose/releases/download/1.4.0/docker-compose-`uname -s`-`uname -m` > /home/core/share/gwydyon/dc/docker-compose && chmod +x /home/core/share/gwydyon/dc/docker-compose", :privileged => true



    end
  end
end