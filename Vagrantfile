# -*- mode: ruby -*-
# vim: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "bootldmachine", primary: true do |bootldmachine|
    bootldmachine.vm.box = "centos/7"
    bootldmachine.vm.box_version = "1804.02"
    bootldmachine.vm.provider "virtualbox" do |v|
        v.gui = true
    end
    bootldmachine.vm.host_name = "bootld-machine"
    bootldmachine.vm.provider "virtualbox" do |v|
        v.memory = "512"
        v.cpus = "2"
    end
    bootldmachine.vm.network "private_network", ip: "192.168.0.32"
    bootldmachine.vm.provision "shell", inline: <<-SHELL
        yum install -y lvm2
    SHELL
  end
end

