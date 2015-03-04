# -*- mode: ruby -*-
# vi: set ft=ruby :

# Builds single Foreman server and
# multiple Puppet Agent Nodes using JSON config file
# Gary A. Stafford - 01/15/2015

# read vm and chef configurations from JSON files
nodes_config = (JSON.parse(File.read("nodes.json")))['nodes']

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true  

  nodes_config.each do |node|
    node_name   = node[0] # name of node
    node_values = node[1] # content of node

    config.vm.define node_name do |config|
      # configures all forwarding ports in JSON array
      if node_values[':mybox'] != nil
        config.vm.box = node_values[':mybox']
      else
        config.vm.box = "chef/centos-6.5"
      end

      ports = node_values['ports']
      ports.each do |port|
        config.vm.network :forwarded_port,
          host:  port[':host'],
          guest: port[':guest'],
          id:    port[':id']
      end

      config.vm.hostname = node_name
      config.vm.network :private_network, ip: node_values[':ip']

      config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", node_values[':memory']]
        vb.customize ["modifyvm", :id, "--name", node_name]
      end

      # synchronize git repo moduels and environemnts in vm
      config.vm.synced_folder "puppet-modules/modules", "/etc/puppet/modules",
        owner: "root", group: "root"
      config.vm.synced_folder "puppet-modules/environments", "/etc/puppet/environments",
        owner: "root", group: "root"

      config.vm.provision :shell, :path => node_values[':bootstrap']
    end
  end
end

