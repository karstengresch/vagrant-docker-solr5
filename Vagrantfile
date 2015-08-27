# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure(2) do |config|

  config.vm.box = "coreos"
  # config.vm.box_url = "http://storage.core-os.net/coreos/amd64-generic/dev-channel/coreos_production_vagrant_vmware_fusion.box"
  config.vm.box_url = "http://storage.core-os.net/coreos/amd64-usr/745.1.0/coreos_production_vagrant_vmware_fusion.json"


  i = 1
  config.vm.define vm_name = 'coreos-solr'
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
  end

  ip = "172.1.1.224"
  config.vm.network :private_network, ip: ip

  config.vm.network "forwarded_port", guest: 8790, host: 14209, auto_correct: false

  config.vm.synced_folder "./", "/home/core/share/",
                          id: "core",
                          nfs_version: "4",
                          :nfs => true,
                          :mount_options => ['nolock,noatime']

  config.vm.provision :shell, :inline => "docker run --name zookeeper -d -p 2181:2181 -p 2888:2888 -p 3888:3888 jplock/zookeeper", :privileged => true
  config.vm.provision :shell, :inline => "docker run --name solr1 --link zookeeper:ZK -d -p 8790:8790 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
  config.vm.provision :shell, :inline => "docker run --name solr2 --link zookeeper:ZK -d -p 8791:8790 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
  config.vm.provision :shell, :inline => "docker run --name solr3 --link zookeeper:ZK -d -p 8792:8790 makuk66/docker-solr bash -c '/opt/solr/bin/solr start -f -z $ZK_PORT_2181_TCP_ADDR:$ZK_PORT_2181_TCP_PORT'", :privileged => true
  config.vm.provision :shell, :inline => 'docker exec -i -t solr1 /opt/solr/bin/solr create_collection -c collection1 -shards 3 -p 8983', :privileged => true



end
