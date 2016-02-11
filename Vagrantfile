# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "s-hironobu/centos67_32_mgtm4pgxc092"
  config.vm.boot_timeout = 500

  config.vm.define :mgtm1 do |mgtm1|
    mgtm1.vm.hostname = "mgtm1"
    mgtm1.vm.network "private_network", ip: "192.168.33.11"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "128"]
    end
  end

  config.vm.define :mgtm2 do |mgtm2|
    mgtm2.vm.hostname = "mgtm2"
    mgtm2.vm.network "private_network", ip: "192.168.33.12"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "128"]
    end
  end

  config.vm.define :mgtm3 do |mgtm3|
    mgtm3.vm.hostname = "mgtm3"
    mgtm3.vm.network "private_network", ip: "192.168.33.13"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "128"]
    end
  end

  config.vm.define :node1 do |node1|
    node1.vm.hostname = "node1"
    node1.vm.network "private_network", ip: "192.168.33.31"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "256"]
    end
  end

  config.vm.define :node2 do |node2|
    node2.vm.hostname = "node2"
    node2.vm.network "private_network", ip: "192.168.33.32"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "256"]
    end
  end
end
