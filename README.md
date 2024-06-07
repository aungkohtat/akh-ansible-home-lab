# akh-ansible-home-lab
![AKH Ansible Home lab](/images/added!.png)

How to Set Up Ansible Home Environment
![Ansible Architecture](/images/akh_absible_home_lab.png)
## Prerequisites
Before we get our hands dirty, let's ensure we have all the necessary tools in our arsenal. You'll need:
- Windows Machine , Mac
- Virtual Box 
- Vagrant
- Code Editor

## Lab Setup
I have already created the necessary Vagrant files and scripts to spin up this lab. You can clone them from my GitHub repository by running the following command:

``` git clonehttps://github.com/aungkohtat/akh-ansible-home-lab.git ```
### The repository contains a Vagrantfile, which looks like this:
``` 
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
```


This Vagrantfile sets up a lab environment with four virtual machines (VMs): a master node, two application nodes, and a database node. Here are the key points:


Base Images:
- Master/Database: "geerlingguy/rockylinux8"
- Application: "ubuntu/jammy64"

Synced Folder: Disabled

Node Configuration:
- Each node has a hostname and private IP.
- Master: 2GB RAM; others: 512MB (adjust as needed).
- App nodes: Run "enable_ssh_password_auth.sh" for SSH password auth.

Node Roles:
- Master: Ansible control.
- App nodes: Web server simulation.
- Database node: Data storage.

## Installing Ansible in the Master Node
``` bash
vagrant up  
```
**fix error**
```
#!/bin/bash

# Enable password auth in sshd so we can use ssh-copy-id
sed -i --regexp-extended 's/#?PasswordAuthentication (yes|no)/PasswordAuthentication yes/' /etc/ssh/sshd_config
sed -i --regexp-extended 's/#?Include \/etc\/ssh\/sshd_config.d\/\*.conf/#Include \/etc\/ssh\/sshd_config.d\/\*.conf/' /etc/ssh/sshd_config
sed -i 's/KbdInteractiveAuthentication no/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd

if [ ! -d /home/vagrant/.ssh ]
then
    mkdir /home/vagrant/.ssh
    chmod 700 /home/vagrant/.ssh
    chown vagrant:vagrant /home/vagrant/.ssh
fi


```

### login master node
``` vagrant ssh master-server ```


This will log you into the master VM we defined earlier. Once you're in, you need to enable the EPEL repository:

``` 
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y 

```

Update the package

```
sudo dnf update
```
Install ansible
```
sudo dnf install ansible -y
```

Once the installation is complete, you can verify the version by running:

```
ansible --version
```

Installing Ansible via PIP
```
sudo dnf update
sudo dnf install python3-pip -y
pip3 install ansible
```

## Set Up SSH Key-Based Authentication for Ansible
```
[vagrant@master-server ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:H5gTceR7Ab1vMchXXshtrLr4MjJFBDspveyMpQtKabM vagrant@master-server
The key's randomart image is:
+---[RSA 3072]----+
|        oo+. . + |
|       . *... o *|
|      . *....o =.|
|       o *..+.= .|
|        S.o .+ o |
|   .   * o.o. o  |
|  = . o o... o   |
| o + . .o + .    |
|  E   .  o +.    |
+----[SHA256]-----+
[vagrant@master-server ~]$ ls -la
total 16
drwx------. 6 vagrant vagrant 139 Jun  7 18:38 .
drwxr-xr-x. 3 root    root     21 Jan  5  2023 ..
drwx------. 4 vagrant vagrant  57 Jan  5  2023 .ansible
-rw-r--r--. 1 vagrant vagrant  18 Apr 12  2022 .bash_logout
-rw-r--r--. 1 vagrant vagrant 141 Apr 12  2022 .bash_profile
-rw-r--r--. 1 vagrant vagrant 376 Apr 12  2022 .bashrc
drwxrwxr-x. 3 vagrant vagrant  17 Jun  7 18:37 .cache
drwx------. 4 vagrant vagrant  28 Jun  7 18:38 .local
drwx------. 2 vagrant vagrant  61 Jun  7 18:45 .ssh
-rw-r--r--. 1 vagrant vagrant   6 Jan  4  2023 .vbox_version
[vagrant@master-server ~]$ ssh-copy-id vagrant@192.168.56.5
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.5 (192.168.56.5)' can't be established.
ECDSA key fingerprint is SHA256:7qAfZ6OYaiKNzceasKd+JhJ4O4WN7mixTgtAljv441w.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@192.168.56.5'"
and check to make sure that only the key(s) you wanted were added.

[vagrant@master-server ~]$ ssh-copy-id vagrant@192.168.56.6
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.6 (192.168.56.6)' can't be established.
ECDSA key fingerprint is SHA256:XnEyIxjelkl/QlDPKuHBOncTReBclrFd+qlruF3zxzs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@192.168.56.6'"
and check to make sure that only the key(s) you wanted were added.

[vagrant@master-server ~]$ ssh-copy-id vagrant@192.168.56.7
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.56.7 (192.168.56.7)' can't be established.
ECDSA key fingerprint is SHA256:jFWCpnqvAHZ/gJmJx19F+eyaxZl0/InyDCRaJWoF4BY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@192.168.56.7's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@192.168.56.7'"
and check to make sure that only the key(s) you wanted were added.

```

## Create ansible config file
On the master node, create a new directory to store our Ansible files:

```
mkdir ~/ansible-lab
```


```
vim ansible.cfg

```
In this file, add the following lines:
```
[defaults]
host_key_checking = False
private_key_file = ~/.ssh/id_rsa

[privilege_escalation]
become=True
```

You can now test the SSH key-based authentication by trying to log in to one of the managed nodes from the master node:

```
ssh vagrant@192.168.56.5
```
```
ssh vagrant@192.168.56.6
```

```
ssh vagrant@192.168.56.7
```

## Create an Ansible Inventory File

```
/etc/ansible/hosts
```
```
vim hosts
```

Add the following lines:

```
192.168.56.4
192.168.56.5
192.168.56.6
```

```
ansible all -i hosts -m ping
```

**Result Output**
```
 192.168.56.5 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.168.56.6 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}


```

you can modify the ansible.cfg

```
[defaults]
inventory = hosts
host_key_checking = False
private_key_file = ~/.ssh/id_rsa

[privilege_escalation]
become=True
```

*******Work Done...!!!*******