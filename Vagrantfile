# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Copyright (c) 2013-2016 Tuomo Tanskanen <tuomo@tanskanen.org>

# read configurable cpu/memory/port/swap/host/edition settings from environment variables
memory = ENV['GITLAB_MEMORY'] || 2048
cpus = ENV['GITLAB_CPUS'] || 1
port = ENV['GITLAB_PORT'] || 8089
swap = ENV['GITLAB_SWAP'] || 0
host = ENV['GITLAB_HOST'] || "gitlab.local"
edition = ENV['GITLAB_EDITION'] || "community"

Vagrant.require_version ">= 1.8.0"

Vagrant.configure("2") do |config|

  # create mgmt node
  config.vm.define :mgmt do |mgmt_config|
      mgmt_config.vm.box = "ubuntu/trusty64"
      mgmt_config.vm.hostname = "mgmt"
      mgmt_config.vm.network :private_network, ip: "10.0.15.10"
      mgmt_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
      mgmt_config.vm.provision :shell, path: "bootstrap-mgmt.sh"
  end

  # create load balancer
  config.vm.define :lb do |lb_config|
      lb_config.vm.box = "ubuntu/trusty64"
      lb_config.vm.hostname = "lb"
      lb_config.vm.network :private_network, ip: "10.0.15.11"
      lb_config.vm.network "forwarded_port", guest: 80, host: 8080
      lb_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
  end

  # create some web servers
  # https://docs.vagrantup.com/v2/vagrantfile/tips.html
  (1..2).each do |i|
    config.vm.define "web#{i}" do |node|
        node.vm.box = "ubuntu/trusty64"
        node.vm.hostname = "web#{i}"
        node.vm.network :private_network, ip: "10.0.15.2#{i}"
        node.vm.network "forwarded_port", guest: 80, host: "808#{i}"
        node.vm.provider "virtualbox" do |vb|
          vb.memory = "256"
        end
    end
  end

  # create gitlab
  config.vm.define :gitlab do |config|
    # Configure some hostname here
    config.vm.hostname = host
    config.vm.box = "ubuntu/xenial64"
    config.vm.provision :shell, :path => "install-gitlab.sh",
      env: { "GITLAB_SWAP" => swap, "GITLAB_HOSTNAME" => host, "GITLAB_PORT" => port, "GITLAB_EDITION" => edition }

    # On Linux, we cannot forward ports <1024
    # We need to use higher ports, and have port forward or nginx proxy
    # or access the site via hostname:<port>, in this case 127.0.0.1:8089
    # By default, Gitlab is at https + port 8443
    config.vm.network :forwarded_port, guest: 80, host: 8089
    config.vm.network :private_network, ip: "10.0.15.66"
  end

  # GitLab recommended specs
  config.vm.provider "virtualbox" do |v, override|
    v.cpus = cpus
    v.memory = memory
  end

  config.vm.provider "vmware_fusion" do |v, override|
    v.vmx["memsize"] = "#{memory}"
    v.vmx["numvcpus"] = "#{cpus}"
    # untested, no vmware license anymore, puppetlabs' vm worked for 14.04
    override.vm.box = "puppetlabs/ubuntu-16.04-64-puppet"
  end

  config.vm.provider "parallels" do |v, override|
    v.cpus = cpus
    v.memory = memory
    # waiting for official "parallels/ubuntu-16.04" vm
    override.vm.box = "boxcutter/ubuntu1604"
  end

  config.vm.provider "lxc" do |v, override|
    override.vm.box = "developerinlondon/ubuntu_lxc_xenial_x64"
  end

  # create jenkins
  config.vm.define :jenkins do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.network "forwarded_port", guest: 80, host: 8088
    config.vm.network :private_network, ip: "10.0.15.55"
    config.vm.provision "shell" do |s|
      s.path = "install-jenkins.sh"
    end
  end
end
