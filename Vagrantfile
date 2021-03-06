# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

@ansible_home = "/home/vagrant/.ansible"

Vagrant.configure("2") do |config|
  config.vm.box = "mpeetani/centos7-latest"

  config.vm.synced_folder "k8cluster/", "#{@ansible_home}/roles/k8cluster", type: 'rsync'
  config.vm.provision "shell", inline: "chown vagrant:vagrant #{@ansible_home}"

  # Kubernetes Master
  config.vm.define "k8smaster" do |master|
    master.vm.hostname = "k8smaster.lab.com"
    master.vm.network "private_network", ip: "172.28.128.10"
    master.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh"
    master.vm.provider "virtualbox" do |vb|
      vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
      vb.name = "k8smaster"
      vb.memory = 4096
      vb.cpus = 2
    end
  end

  nodecount = 2

  # Kubernetes Worker Nodes
  (1..nodecount).each do |i|
    config.vm.define "k8snode#{i}" do |node|
      node.vm.hostname = "k8snode#{i}.lab.com"
      node.vm.network "private_network", ip: "172.28.128.4#{i}"
      node.vm.network :forwarded_port, guest: 22, host: "220#{i}", id: "ssh"
      node.vm.provider "virtualbox" do |vb|
       vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
       vb.name = "k8snode#{i}"
       vb.memory = 2048
       vb.cpus = 1
      end
    end
  end
# Copy host user ssh keys to the vagrant VMs

  id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))
    config.vm.provision :shell,
       :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"

# Update servers to latest OS versions

config.vm.provision "shell", inline: <<-SHELL
    yum update -y
    yum install -y lvm2 wget git vim epel-release curl python3
    SHELL

# Install K8s on master and worker Nodes
control.vm.provision "ansible_local" do |ansible|
       ansible.playbook = "k8cluster.yml"
       ansible.inventory_path = "inventory"
       ansible.limit = "all"
end
end
