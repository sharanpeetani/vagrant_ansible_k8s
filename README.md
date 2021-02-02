# kubernetes-cluster

Initialize a Kubernetes cluster with Prometheus and Grafana for monitoring using ansible.


# Ansible Host file

You need to create a static inventory file that corresponds to your Vagrantfile machine definitions:


```
masternode ansible_host=127.0.0.1 ansible_port=2222 ansible_user='vagrant' ansible_ssh_private_key_file='/mnt/d/DevOpsProjects/k8Cluster/kubernetes-cluster/.vagrant/machines/masternode/virtualbox/private_key'
node1 ansible_host=127.0.0.1 ansible_port=2200 ansible_user='vagrant' ansible_ssh_private_key_file='/mnt/d/DevOpsProjects/k8Cluster/kubernetes-cluster/.vagrant/machines/node1/virtualbox/private_key'
node2 ansible_host=127.0.0.1 ansible_port=2201 ansible_user='vagrant' ansible_ssh_private_key_file='/mnt/d/DevOpsProjects/k8Cluster/kubernetes-cluster/.vagrant/machines/node2/virtualbox/private_key'

[nodes]
node[1:2]

[master]
masternode

```

# ansible.cfg

Create an ansible.cfg file to fully disable SSH host key checking

```
[defaults]
host_key_checking = no

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes

```


# Vagrantfile

Spin up 3 CentOS 7 nodes; one master and two worker nodes

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

#@ansible_home = "/home/vagrant/.ansible"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  # Kubernetes Master
  config.vm.define "masternode" do |master|
    master.vm.hostname = "masternode"

    master.vm.network "private_network", ip: "172.28.128.10"
       
    master.vm.provider "virtualbox" do |vb|
      vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
      vb.name = "masternode"
      vb.memory = 2048
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
      end  
    end	
  end
end

```


# Usage

From the project folder, run the following commands, one after the other.

1. Start up the machines:

`vagrant up`

2. Run ansibble to initialize and provision the cluster

`ansible-playbook --limit="all" --inventory-file=inventory k8cluster.yml`
