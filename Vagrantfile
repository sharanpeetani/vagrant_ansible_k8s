# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

@ansible_home = "/home/vagrant/.ansible"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.synced_folder "k8cluster/", "#{@ansible_home}/roles/k8cluster", type: 'rsync'
  config.vm.provision "shell", inline: "chown vagrant:vagrant #{@ansible_home}"

  # Kubernetes Master
  config.vm.define "masternode" do |master|
    master.vm.hostname = "masternode"
    master.vm.network "private_network", ip: "172.28.128.10"      
    master.vm.provider "virtualbox" do |vb|
      vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
      vb.name = "masternode"
      vb.memory = 4096
      vb.cpus = 2
    end    
  end
  
  nodecount = 2
  
  # Kubernetes Worker Nodes 
  (1..nodecount).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.hostname = "node#{i}"
      node.vm.network "private_network", ip: "172.28.128.4#{i}"

      node.vm.provider "virtualbox" do |vbw|
       vbw.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
       vbw.name = "node#{i}" 
       vbw.memory = 2048
       vbw.cpus = 1
      end  
    end	
  end
end
