# Puppet with Vagrant for local testing

- [Puppet with Vagrant for local testing](#puppet-with-vagrant-for-local-testing)
  - [Installation](#installation)
    - [Install vagrant](#install-vagrant)
    - [Copy the files for VM creation and provisioning](#copy-the-files-for-vm-creation-and-provisioning)
  - [Managing the VM with Vagrant](#managing-the-vm-with-vagrant)
    - [Start the VM and opening the SSH session](#start-the-vm-and-opening-the-ssh-session)
    - [Stop the VM](#stop-the-vm)
    - [Delete the VM](#delete-the-vm)
    - [Add a shared folder for a Windows host](#add-a-shared-folder-for-a-windows-host)
  - [Running a simple manifest](#running-a-simple-manifest)
  - [Running a custom environment](#running-a-custom-environment)
    - [Running a custom environment with the 'roles and profiles' method](#running-a-custom-environment-with-the-roles-and-profiles-method)
  - [Puppet conventions](#puppet-conventions)
    - [The confdir](#the-confdir)
    - [Settings for specific Sections in puppet.conf](#settings-for-specific-sections-in-puppetconf)
    - [The default environmentpath](#the-default-environmentpath)
    - [Manifest directory behavior](#manifest-directory-behavior)
    - [The modulepath and the base modulepath](#the-modulepath-and-the-base-modulepath)
  - [Troubleshooting Puppet](#troubleshooting-puppet)
    - [The module repository](#the-module-repository)
    - [Get configuration values in the node](#get-configuration-values-in-the-node)
    - [Get fact values in the node](#get-fact-values-in-the-node)
    - [Override facts using 'Environment facts'](#override-facts-using-environment-facts)
    - [See what the modulepath is for an environment](#see-what-the-modulepath-is-for-an-environment)
    - [Check hiera data values](#check-hiera-data-values)
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
mkdir -p ./vms/puppet-test
cp $PROJECT/{Vagrantfile.template,provision.sh} ./vms/puppet-test
cd ./vms/puppet-test
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

## Running a simple manifest

```sh
echo 'notice("Hello World")' > hello.pp
puppet apply hello.pp
```

## Running a custom environment

**1 - Create a temp environments with your example environment**

```sh
mkdir -p ./example-envs/mycustomenv
```

**2 - Check defaults**

The output of the following command shows that, even when you didn't created the environment.conf with the modulepath setting, the default modulepath value for an environment is the environment's modules directory, plus the base modulepath. On *nix, this is './modules:$basemodulepath'.

```sh
puppet config print modulepath \
--section user \
--environment mycustomenv \
--environmentpath ./example-envs

# Example response
# /home/vagrant/example-envs/mycustomenv/modules:/home/vagrant/.puppetlabs/etc/code/modules:/opt/puppetlabs/puppet/modules
```

The following command shows the main manifest.

NOTE: './manifests' means \<ENVIRONMENT\>/manifests. That is, the directory called 'manifests' inside the directory of the selected environment.

```sh
puppet config print default_manifest \
--section user \
--environment mycustomenv \
--environmentpath ./example-envs

# Example response
# ./manifests
```

**3 - Create the expected dafaults**

```sh
mkdir -p ./example-envs/mycustomenv/{modules,manifests}
```

**4 - Create the manifests**

```sh
cat <<EOT > ./example-envs/mycustomenv/manifests/site.pp
node default {
  class { 'entrypoint': }
}
EOT

cat <<EOT > ./example-envs/mycustomenv/manifests/entrypoint.pp
class entrypoint {
  notice("Hello from environment: \${environment}")
  notice("Puppet version: \${puppetversion}")
  notice("Ruby version: \${facts['ruby']['version']}")
  notice("Environment: \${environment}")
  notice("Module path: \${settings::modulepath}")
}
EOT

cat <<EOT > ./example-envs/mycustomenv/manifests/donothing.pp
class donothing {
  notice("This message will not be printed!")
}
EOT

```

**3 - Run the environment**

Run the manifests directory with the 'node' declarations:

```sh
puppet apply ./example-envs/mycustomenv/manifests \
--environment mycustomenv \
--environmentpath ./example-envs
```

### Running a custom environment with the 'roles and profiles' method

Whit this project structure, te environment is composed of modules (role and profile).

The 'role' module is the module that stores different manifests, classes that we will use to manage different thinks. Each class in the role module is independent and represents a job to run in a given moment.

The 'profile' module stores manifests declaring classes for multiple tasks that can being used by the roles.

See [The roles and profiles method](https://puppet.com/docs/pe/2021.0/osp/the_roles_and_profiles_method.html) for details.

**1 - Create a temp environments with your example project**

```sh
mkdir -p ./example-envs/mycustomenv2
```

**2 - Create the basic project structure**

```sh
cat <<EOT > ./example-envs/mycustomenv2/hiera.yaml
---
version: 5
defaults:
  datadir: "hieradata"
  data_hash: yaml_data

hierarchy:
  - name: "environment"
    path: "environments/%{::environment}.yaml"

  - name: "common"
    path: "common.yaml"
EOT

mkdir -p ./example-envs/mycustomenv2/hieradata/environments

cat <<EOT > ./example-envs/mycustomenv2/hieradata/common.yaml
---
message: "This node is using common data"
EOT

cat <<EOT > ./example-envs/mycustomenv2/hieradata/environments/mycustomenv2.yaml
---
profile::printinfo::message: "%{hiera('message')} and custom environment data."
EOT

mkdir -p ./example-envs/mycustomenv2/site-modules/{profile,role}/manifests

# 'site-modules': this directory stores our 'role' and 'profile' modules.
# 'modules': this directory stores the downloaded dependencies/modules with r10k, as shown later.
cat <<EOT > ./example-envs/mycustomenv2/environment.conf
modulepath = site-modules:modules:$basemodulepath
EOT

cat <<EOT > ./example-envs/mycustomenv2/Puppetfile
# modules from Puppet Forge
mod 'puppetlabs/stdlib', '4.24.0'
mod 'puppetlabs/docker', '3.5.0'
mod 'puppetlabs/java', '3.3.0'
EOT

cat <<EOT > ./example-envs/mycustomenv2/site-modules/profile/manifests/printinfo.pp
class profile::printinfo (
  String \$message
) {
  notice("Hello from environment: \${environment}")
  notice("Puppet version: \${puppetversion}")
  notice("Ruby version: \${facts['ruby']['version']}")
  notice("Environment: \${environment}")
  notice("Module path: \${settings::modulepath}")
  notice("Message: \${message}")
}
EOT

cat <<EOT > ./example-envs/mycustomenv2/site-modules/role/manifests/main.pp
class role::main {
  include profile::printinfo
}
EOT
```

**3 - Install the dependencies**

First we need to install 'r10k'. See [r10k - Smarter Puppet deployment](https://github.com/puppetlabs/r10k) for details.

```sh
sudo /opt/puppetlabs/puppet/bin/gem install r10k
/opt/puppetlabs/puppet/bin/r10k help
```

This command will be downloading the dependencies into the directory 'modules' as a sibling of the Puppetfile file.

```sh
cd ./example-envs/mycustomenv2
/opt/puppetlabs/puppet/bin/r10k puppetfile install
cd ../..
```

**4 - Run the project**

With the 'roles and profiles' method you can invoke an specific class of the role module directly without a 'node' declaration:

```sh
puppet apply -e "include role::main" \
--environment mycustomenv2 \
--environmentpath ./example-envs
```

## Puppet conventions

### The confdir

The puppet confdir depends on user that executes puppet command. That's why puppet cofiguration will differ for different users.

Puppet's confdir is the main directory for the Puppet configuration. It contains configuration files and the SSL data.

The confdir is located in one of the following locations:
- *nix root users: /etc/puppetlabs/puppet
- Non-root users: ~/.puppetlabs/etc/puppet
- Windows: %PROGRAMDATA%\PuppetLabs\puppet\etc (usually C:\ProgramData\PuppetLabs\puppet\etc)

When Puppet is running as root, a Windows user with administrator privileges, or the puppet user, it uses a system-wide confdir. When running as a non-root user, it uses a confdir in that user's home directory.

When running Puppet commands and services as root or puppet, usually you want to use the system codedir. To use the same codedir as the Puppet agent or the primary Puppet server, run admin commands with sudo.

Puppet's confdir can't be set in the puppet.conf, because Puppet needs the confdir to locate that config file.

See [Config directory (confdir)](https://puppet.com/docs/puppet/7/dirs_confdir.html) for details.

### Settings for specific Sections in puppet.conf

puppet.conf resembles a standard INI file, with a few syntax extensions. Settings can go into application-specific sections, or into a ´'\[main\]' section that affects all applications.

See [rightpuppet.conf: The main config file](https://puppet.com/docs/puppet/7/config_file_main.html) for details.

For the 'puppet config print' command, we can use the --section option to specify which section of puppet.conf to use when finding settings. It is optional, and defaults to main.

Valid sections are:
- main (default) — used by all commands and services.
- server - used by the primary Puppet server service and the 'puppetserver ca' command.
- agent - used by the Puppet agent service.
- user - used by the 'puppet apply' command and most other commands.

As usual, the other sections override the main section if they contain a setting; if they don't, the value from main is used, or a default value if the setting isn't present there.

See [Checking the values of settings](https://puppet.com/docs/puppet/7/config_print.html) for details.

### The default environmentpath

The environmentpath setting is the list of directories where Puppet looks for environments. The default value for environmentpath is $codedir/environments. If you have more than one directory, separate them by colons and put them in order of precedence.

In this example, temp_environments is searched before environments: $codedir/temp_environments:$codedir/environments

If environments with the same name exist in both paths, Puppet uses the first environment with that name that it encounters.

Put the environmentpath setting in the main section of the puppet.conf file.

See [Creating environments](https://puppet.com/docs/puppet/7/environments_creating.html) for details.

### Manifest directory behavior

When the main or default manifest is a directory, Puppet parses every .pp file in the directory in alphabetical order and evaluates the combined manifest. It descends into all subdirectories of the manifest directory and loads files in depth-first order. For example, if the manifest directory contains a directory named 01, and a file named 02.pp, it parses the files in 01 before it parses 02.pp.

Puppet treats the directory as one manifest, so, for example, a variable assigned in the file 01_all_nodes.pp is accessible in node_web01.pp.

See: [Main manifest directory](https://puppet.com/docs/puppet/7/dirs_manifest.html) for details.

### The modulepath and the base modulepath

The modulepath can include relative paths, such as ./modules or ./site. Puppet looks for these paths inside the environment's directory.

The default modulepath value for an environment is the environment's modules directory, plus the base modulepath. On *nix, this is './modules:$basemodulepath'.

The base modulepath is a list of global module directories for use with all environments. You can configure it with the basemodulepath setting in the puppet.conf file, but its default value is probably suitable. The default on *nix is '$codedir/modules:/opt/puppetlabs/puppet/modules'.

See [The modulepath](https://puppet.com/docs/puppet/7/dirs_modulepath.html) for details.

## Troubleshooting Puppet

### The module repository

By default the module repository is Puppet Forge:

```sh
puppet config print module_repository
# result:
#   https://forgeapi.puppet.com
```

You can change this by modifying the 'puppet.conf' file:

```sh
vi /etc/puppetlabs/puppet/puppet.conf
```

Example:

```ini
[main]
module_repository=http://localhost:8080/
```

### Get configuration values in the node

For global settings:

```sh
puppet config print
```

For settings in 'puppet apply':

```sh
puppet config print --section user
```

### Get fact values in the node

Before requesting a catalog for a managed node, or compiling one with `puppet apply`, Puppet collects system information, called 'facts', by using the Facter tool. The facts are assigned as values to variables that you can use anywhere in your manifests.

NOTE: Puppet also sets some additional special variables, called built-in variables, which behave a lot like facts.

NOTE: Puppet honors fact values of of any data type. It does not convert Boolean, numeric, or structured facts to strings.

You could:
- Run the command `facter -p` to get all the facts,
- or run `facter <FACT_NAME>` to get the value of a fact,
- or browse facts on node detail pages in the Puppet Enterprise console.

### Override facts using 'Environment facts'

Environment facts allow you to override core facts and add custom facts.

Use the environment variable prefixed with 'FACTER_', for example:

```sh
facter virtual
# result:
#   physical

FACTER_virtual=virtualbox facter virtual
# result:
#   virtualbox
```

NOTE: Note that environment facts are downcased before they are added to the fact collection, for example, FACTER_EXAMPLE and FACTER_example resolve to a single fact named example.

With this feature you could provide custom values when running a job with 'puppet apply'. Example:

```sh
FACTER_mypassword=123456 \
puppet apply -e "include role::main" \
--environment mycustomenv2 \
--environmentpath ./example-envs
```

### See what the modulepath is for an environment

For a node in a puppet master architecture:

```sh
puppet config print modulepath --section server --environment <ENVIRONMENT_NAME>
```

For a node using 'puppet apply':

```sh
puppet config print modulepath --section user --environment <ENVIRONMENT_NAME>
```

For a node using 'puppet apply' with environment directory not located in default '$codedir/environments/' (configured in the 'environmentpath' setting) where Puppet looks for environments:

```sh
puppet config print modulepath --section user --environmentpath <ENVIRONMENTS_DIR> --environment <ENVIRONMENT_NAME>
```

### Check hiera data values

The following examples checks the value of 'profile::printinfo::message'.

**Alternative 1 - Use the puppet lookup command**

```sh
puppet lookup profile::printinfo::message --environment mycustomenv2 --environmentpath ./example-envs
```

**Alternative 2 - Use the lookup function**

The following example checks the value of 'profile::printinfo::message'.

```sh
puppet apply -e "notice(lookup(profile::printinfo::message))" --environment mycustomenv2 --environmentpath ./example-envs
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
sudo yum -y install wget curl vim bash-completion

# Add the epel repository
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Add the repository for latest Puppet release on CentOS 7|RHEL 7
# See http://yum.puppet.com/ for a list of available RPM repositories
sudo yum -y install https://yum.puppet.com/puppet-release-el-7.noarch.rpm

# Install the Puppet agent package
sudo yum -y install puppet-agent
# Install r10k gem to be able to install modules/dependencies from Puppetfile
sudo /opt/puppetlabs/puppet/bin/gem install r10k
```
