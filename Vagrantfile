# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

NUM_OF_NODES = 1

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "cbednarski/ubuntu-1404"
  # config.vm.box = "ubuntu/trusty64"
  provisioner = Vagrant::Util::Platform.windows? ? :guest_ansible : :ansible

  # Consul server. A significantly influenced by: https://github.com/trydb/tryconsul
  config.vm.define "consul" do |c1|
    c1.vm.network "private_network", ip: "10.7.0.15"
    c1.vm.provision "shell", inline: "hostnamectl set-hostname consul"
    c1.vm.provision "shell", inline: "cd /vagrant/consul && make deps install install-server"
    c1.vm.provision "shell", inline: "hostess add consul $(</tmp/self.ip)"
    config.vm.provider "virtualbox" do |v|
      v.memory = 256
      v.cpus = 1
    end
  end

  # Nomad server
  config.vm.define "nomad" do |n|
    n.vm.network "private_network", ip: "10.7.0.10"
    n.vm.provision "shell", inline: "hostnamectl set-hostname nomad"
    n.vm.provision "shell", inline: "cd /vagrant/consul && make install install-client" # install consul
    n.vm.provision "shell", inline: "consul join 10.7.0.15"
    n.vm.provision "shell", inline: "cd /vagrant/nomad && make deps install install-server"
    n.vm.provision "shell", inline: "hostess add nomad $(</tmp/self.ip)"
    config.vm.provider "virtualbox" do |v|
      v.memory = 256
      v.cpus = 1
    end
  end

  # Nomad client / Docker hosts
  (1..3).each do |d|
    config.vm.define "docker#{d}" do |node|
      node.vm.network "private_network", ip: "10.7.0.2#{d}" # 10.7.0.21, 10.7.0.22, 10.7.0.23
      node.vm.provision "shell", inline: "hostnamectl set-hostname docker#{d}"
      node.vm.provision "shell", inline: "cd /vagrant/consul && make install install-client" # install consul-client
      node.vm.provision "shell", inline: "consul join 10.7.0.15"
      node.vm.provision "docker" # Just install it
      node.vm.provision "shell", inline: "cd /vagrant/nomad && make install install-client"
      node.vm.provision "shell", inline: "hostess add docker#{d} $(</tmp/self.ip)"
      config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
      end
    end
  end

  # HAProxy server. Taken from: https://github.com/foostan/consul-with-docker
  (1..NUM_OF_NODES).each do |d|
    config.vm.define "haproxy#{d}" do |node|
      node.vm.network "private_network", ip: "10.7.0.3#{d}" # 10.7.0.31
      node.vm.provision "shell", inline: "hostnamectl set-hostname haproxy#{d}"
      node.vm.provision "shell", inline: "cd /vagrant/consul && make install install-client" # install consul-client
      node.vm.provision "shell", inline: "consul join 10.7.0.15"
      node.vm.provision "shell", inline: "cd /vagrant/nomad && make install install-client"
      node.vm.provision "shell", inline: "hostess add haproxy#{d} $(</tmp/self.ip)"
      node.vm.provision provisioner do |ansible|
        ansible.playbook = "playbook.yml"
      end
    end
  end

end



