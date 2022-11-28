# Docker with Vagrant for local development

- [Docker with Vagrant for local development](#docker-with-vagrant-for-local-development)
  - [Installation](#installation)
    - [Install vagrant](#install-vagrant)
    - [Copy the files for VM creation and provisioning](#copy-the-files-for-vm-creation-and-provisioning)
  - [Managing the VM with Vagrant](#managing-the-vm-with-vagrant)
    - [Start the VM and opening the SSH session](#start-the-vm-and-opening-the-ssh-session)
    - [Stop the VM](#stop-the-vm)
    - [Delete the VM](#delete-the-vm)
  - [Running a simple container](#running-a-simple-container)
  - [Docker conventions](#docker-conventions)
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

## Running a simple container

TODO

## Docker conventions

TODO

## Troubleshooting Docker

TODO

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
# { "insecure-registries" : ["myartifactory.local:8081"] }
# EOT

# Install docker-compose
sudo curl \
  -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
