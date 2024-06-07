# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #config.ssh.insert_key = false

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.boot_timeout = 900

  # master server.
  config.vm.define "master-server" do |master|
    master.vm.box = "geerlingguy/rockylinux8"
    master.vm.hostname = "master-server"
    master.vm.network :private_network, ip: "192.168.56.4"
    master.vm.provider "virtualbox" do |v|
      v.memory = 2048 # 2GB RAM
    end
  end

  # Application server 1.
  config.vm.define "app01" do |app|
    app.vm.box = "ubuntu/jammy64"
    app.vm.hostname = "app-server-01"
    app.vm.network :private_network, ip: "192.168.56.5"
    app.vm.provider "virtualbox" do |v|
      v.memory = 512 # 1GB RAM
    end
    app.vm.provision "shell", path: "enable_ssh_password_auth.sh"
  end

  # Application server 2.
  config.vm.define "app02" do |app|
    app.vm.box = "ubuntu/jammy64"
    app.vm.hostname = "app-server-02"
    app.vm.network :private_network, ip: "192.168.56.6"
    app.vm.provider "virtualbox" do |v|
      v.memory = 512 # 1GB RAM
    end
    app.vm.provision "shell", path: "enable_ssh_password_auth.sh"
  end

  # Database server 1.
  config.vm.define "db" do |db|
    db.vm.box = "geerlingguy/rockylinux8"
    db.vm.hostname = "db-server-01"
    db.vm.network :private_network, ip: "192.168.56.7"
    db.vm.provider "virtualbox" do |v|
      v.memory = 512 # 1GB RAM
    end
  end

end
