# Docker with Vagrant for local development

- [Docker with Vagrant for local development](#docker-with-vagrant-for-local-development)
  - [Installation](#installation)
    - [Install vagrant](#install-vagrant)
    - [Copy the files for VM creation and provisioning](#copy-the-files-for-vm-creation-and-provisioning)
  - [Managing the VM with Vagrant](#managing-the-vm-with-vagrant)
    - [Start the VM and opening the SSH session](#start-the-vm-and-opening-the-ssh-session)
    - [Stop the VM](#stop-the-vm)
    - [Delete the VM](#delete-the-vm)
    - [Add a shared folder for a Windows host](#add-a-shared-folder-for-a-windows-host)
  - [Get images from another registry](#get-images-from-another-registry)
    - [Option 1 - Login into a secured registry](#option-1---login-into-a-secured-registry)
    - [Option 2 - Add an insecure registry](#option-2---add-an-insecure-registry)
  - [Troubleshooting Docker](#troubleshooting-docker)
  - [Project files](#project-files)

## Installation

Open a administrative bash shell and follow the steps bellow.

NOTE: I'll be using the bash shell provided by my installation of git on Windows.

### Install vagrant

1. Download the new package from `https://www.vagrantup.com/downloads`.
2. Install it (even an old installation because it is like updating it).
3. Update the provider VirtualBox: `vagrant box update`.
4. Install the required plugins for vagrant:
```sh
# VirtualBox Guest Additions up to date (winnfsd requires vbguest 0.29.0)
vagrant plugin install vagrant-vbguest --plugin-version=0.29.0
# NFS support for Windows
vagrant plugin install vagrant-winnfsd
# Manages the 'hosts' file on guest machines (and optionally the host).
vagrant plugin install vagrant-hostmanager
```

### Copy the files for VM creation and provisioning

```sh
cd $USERPROFILE
mkdir -p ./vms/docker-test
cp $PROJECT/{Vagrantfile.template,provision.sh} ./vms/docker-test
cd ./vms/docker-test
mv Vagrantfile.template Vagrantfile
```

## Managing the VM with Vagrant

### Start the VM and opening the SSH session

```sh
vagrant up
vagrant ssh
```

### Stop the VM

```sh
vagrant halt
```

### Delete the VM

```sh
vagrant destroy
```

### Add a shared folder for a Windows host

**Create the folder as a sibling of Vagrantfile**

```sh
mkdir shared
```

**Add the following into the 'Vagrant.configure' block**

```ruby
# NFS requires a host-only network to be created.
# We need to add a host-only network to the machine (with either DHCP or a static IP) for NFS to work.
config.vm.network "private_network", ip: "192.168.33.10"

config.vm.synced_folder '.', '/vagrant', disabled: true
config.vm.synced_folder "./shared", "/var/shared",
  type:"nfs",
  create: true,
  # group:'nginx', owner:'magento', # no supported by NFS
  mount_options: %w{rw,async,fsc,nolock,vers=3,udp,rsize=32768,wsize=32768,hard,noatime,actimeo=2}
```

**Releoad the VM with the modifications**

You can make `vagrant reload --no-provision` or the following:

```sh
vagrant halt
vagrant up --no-provision
```

Note I use '--no-provision' to avoid re-provisioning, that is to avoid re-executing the provision.sh script.

**Connect to the guest (the VM)**

```sh
# Connect to the guest
vagrant ssh
```

**NFS service and access in the guest (the VM)**

```sh
# Enable the NFS service
sudo systemctl enable nfs
# Enable serving files from NFS mounts:
sudo setsebool -P httpd_use_nfs 1
```

## Get images from another registry

By default, all images will be pulled from  the 'Docker Hub' registry. If you want to download an images from other locations, for example artifactory, then you have to follow one of the following options.

### Option 1 - Login into a secured registry

First, you have to login into the registry.

Example:

```sh
sudo docker login artifactory.dev.local
# username: cmuser
# password: Palindrome10
```

Now you can pull images using this registry.

In this example I have my images in the 'docker-local' location:

```sh
sudo docker pull artifactory.dev.local/docker-local/myproduct/origin/master/myproduct:latest
sudo docker pull artifactory.dev.local/docker-local/common/nginx:v1.0.3.BUILD-1
```

### Option 2 - Add an insecure registry

For insecure sources, you have to add the registry into the file daemon.json.

```sh
sudo tee /etc/docker/daemon.json > /dev/null <<EOT
{ "insecure-registries" : ["myartifactory.local:8081"] }
EOT
```

## Troubleshooting Docker

**Interactive shell into the running container**

```sh
sudo docker exec -it <CONTAINER> bash
```

## Project files

**Vagrantfile.template**

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  # Required for centos/7 or centos/8
  # Alternative 1: Install the latest kernel and perform a reboot of the Vagrant VM before continuing the provisioning.
  config.vbguest.installer_options = { allow_kernel_upgrade: true }

  config.vm.provision "shell" do |s|
    s.path = "provision.sh"
  end
end
```

**provision.sh**

```sh
set -x #echo on

# Update the system
sudo yum -y update

# Install required packages
sudo yum install -y yum-utils

# Add the docker-ce repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install docker
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker
sudo systemctl start docker

# Configure your custom repositories to download images
# sudo tee /etc/docker/daemon.json > /dev/null <<EOT
# { "insecure-registries" : ["artifactory.dev.local:8081"] }
# EOT

# Install docker-compose
sudo curl \
  -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
