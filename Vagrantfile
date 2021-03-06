# -*- mode: ruby -*-
# vi: set ft=ruby :

# Every Vagrant development environment requires a box. You can search for
# boxes at https://vagrantcloud.com/search.
BOX_IMAGE = "ubuntu/bionic64"

# Networking
HOST_ONLY_NETWORK = "192.168.68"
LB_IP = 5
MASTER_IP_START_AT = 10
WORKER_IP_START_AT = 20

# Spec
MASTER_NODES = 3
MASTER_CPU = 2
MASTER_RAM = 2048

WORKER_NODES = 3
WORKER_CPU = 2
WORKER_RAM = 2048

LB_CPU = 1
LB_RAM = 2048


Vagrant.configure("2") do |config|
  # LB node
  config.vm.define "lb01" do |l|
    l.vm.provider "virtualbox" do |v|
      v.name = "k8s-lb01"
      v.cpus = LB_CPU
      v.memory = LB_RAM
    end

    l.vm.box = BOX_IMAGE
    l.vm.hostname = "lb01"
    l.vm.network "private_network", ip: "#{HOST_ONLY_NETWORK}.#{LB_IP}"

    l.vm.provision :salt do |s|
      s.minion_config = "saltstack/etc/lb01"
      s.minion_key = "saltstack/keys/lb01.pem"
      s.minion_pub = "saltstack/keys/lb01.pub"
      s.install_type = "stable"
      s.colorize = true
      s.verbose = true
      s.bootstrap_options = "-P -c /tmp"
    end
  end

  $disableSwap = <<-'SCRIPT'
    swapoff -a
    sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
  SCRIPT
  # Master nodes
  (1..MASTER_NODES).each do |i|
    config.vm.define "master#{i}" do |m|
      m.vm.provider "virtualbox" do |v|
        v.name = "k8s-master#{i}"
        v.cpus = MASTER_CPU
        v.memory = MASTER_RAM
      end

      m.vm.box = BOX_IMAGE
      m.vm.hostname = "master#{i}"
      m.vm.network "private_network", ip: "#{HOST_ONLY_NETWORK}.#{MASTER_IP_START_AT + i}"
      m.vm.provision "shell", inline: $disableSwap

      m.vm.provision :salt do |s|
        s.minion_config = "saltstack/etc/master#{i}"
        s.minion_key = "saltstack/keys/master#{i}.pem"
        s.minion_pub = "saltstack/keys/master#{i}.pub"
        s.install_type = "stable"
        s.colorize = true
        s.verbose = true
        s.bootstrap_options = "-P -c /tmp"
      end
    end
  end

  # Worker nodes
  (1..WORKER_NODES).each do |i|
    config.vm.define "worker#{i}" do |w|
      w.vm.provider "virtualbox" do |v|
        v.name = "k8s-worker#{i}"
        v.cpus = WORKER_CPU
        v.memory = WORKER_RAM
      end

      w.vm.box = BOX_IMAGE
      w.vm.hostname = "worker#{i}"
      w.vm.network "private_network", ip: "#{HOST_ONLY_NETWORK}.#{WORKER_IP_START_AT + i}"
      w.vm.provision "shell", inline: $disableSwap

      w.vm.provision :salt do |s|
        s.minion_config = "saltstack/etc/worker#{i}"
        s.minion_key = "saltstack/keys/worker#{i}.pem"
        s.minion_pub = "saltstack/keys/worker#{i}.pub"
        s.install_type = "stable"
        s.colorize = true
        s.verbose = true
        s.bootstrap_options = "-P -c /tmp"
      end
    end
  end
end