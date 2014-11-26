# -*- mode: ruby -*-
# vi: set ft=ruby :
#John R. Ray
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos-65-x64-nocm"
  config.vm.box_url = "http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-nocm.box"

  ## Server
  config.vm.define :server do |s|
    s.vm.provider :virtualbox do |v|
      v.memory  = 1024
      v.cpus = 2
    end
    s.vm.network :private_network,  ip: "10.10.100.100"
    s.vm.hostname = 'server.shadow-soft.com'
    s.vm.provision :hosts
  end

  ## Client 
  config.vm.define :client do |c|
    c.vm.provider :virtualbox
    c.vm.network :private_network,  ip: "10.10.100.111"
    c.vm.hostname = 'client.shadow-soft.com'
    c.vm.provision :hosts
  end
end
