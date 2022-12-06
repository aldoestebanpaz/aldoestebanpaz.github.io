# Ansible with Vagrant for local testing

- [Ansible with Vagrant for local testing](#ansible-with-vagrant-for-local-testing)
  - [Installation](#installation)
    - [Install vagrant](#install-vagrant)
    - [Copy the files for VM creation and provisioning](#copy-the-files-for-vm-creation-and-provisioning)
  - [Managing the VM with Vagrant](#managing-the-vm-with-vagrant)
    - [Start the VM and opening the SSH session](#start-the-vm-and-opening-the-ssh-session)
    - [Stop the VM](#stop-the-vm)
    - [Delete the VM](#delete-the-vm)
    - [Add a shared folder for a Windows host](#add-a-shared-folder-for-a-windows-host)
  - [Running ansible code from command line](#running-ansible-code-from-command-line)
    - [Running shell commands with ansible](#running-shell-commands-with-ansible)
    - [Passing non-raw parameters to the module](#passing-non-raw-parameters-to-the-module)
  - [Running a simple playbook locally](#running-a-simple-playbook-locally)
  - [Running an advanced playbook with other ansible artifacts](#running-an-advanced-playbook-with-other-ansible-artifacts)
    - [Creating and running the playbook](#creating-and-running-the-playbook)
    - [Understanding the concepts behind some basic ansible artifacts](#understanding-the-concepts-behind-some-basic-ansible-artifacts)
      - [Play](#play)
      - [Playbook](#playbook)
      - [The Role file structure](#the-role-file-structure)
      - [Variables and variables file](#variables-and-variables-file)
      - [Tasks and tasks file](#tasks-and-tasks-file)
      - [Tags](#tags)
      - [Handlers and handlers file](#handlers-and-handlers-file)
  - [Understanding variables](#understanding-variables)
    - [Variables discovered from nodes (facts)](#variables-discovered-from-nodes-facts)
    - [Custom facts](#custom-facts)
    - [Custom variables](#custom-variables)
    - [How to get nested values of data structures](#how-to-get-nested-values-of-data-structures)
  - [Using collections](#using-collections)
  - [Troubleshooting](#troubleshooting)
    - [Verifying playbooks](#verifying-playbooks)
      - [Syntax check](#syntax-check)
      - [List available tags](#list-available-tags)
      - [List tasks that would be executed](#list-tasks-that-would-be-executed)
      - [Dry-run (aka. simulate)](#dry-run-aka-simulate)
      - [Linter](#linter)
    - [Get fact values locally](#get-fact-values-locally)
    - [List installed collections](#list-installed-collections)
    - [Ansible doesn't see an installed Python module when localhost is specified in the inventory](#ansible-doesnt-see-an-installed-python-module-when-localhost-is-specified-in-the-inventory)
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

## Running ansible code from command line

Running ansible code from command line, that is a quick one-liner in Ansible without writing a playbook, is called an 'ad hoc command'. An Ansible ad hoc command uses the /usr/bin/ansible command-line tool to automate a single task.

See [Introduction to ad hoc commands](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html) for details.

### Running shell commands with ansible

The default module for the 'ansible' command is the 'ansible.builtin.command' module. This module takes the command name followed by a list of space-delimited arguments.

NOTE: Modules like 'command' and 'shell' has the raw attribute. This indicates that an action can take a raw or free form string as an option and has it's own special parsing of it. This means that '-a' in these cases, accepts free form strings in these cases.

```sh
ansible localhost -a "echo Hello World"
# result:
#   localhost | CHANGED | rc=0 >>
#   Hello World

ansible localhost -a "echo {{ ansible_version.full }}"
# result:
#   localhost | CHANGED | rc=0 >>
#   2.9.27
```

NOTE: The 'command' module does not support extended shell syntax like piping and redirects (although shell variables will always work). If your command requires shell-specific syntax, use the 'shell' module instead. Read more about the differences on the [Working With Modules](https://docs.ansible.com/ansible/6/user_guide/modules.html#working-with-modules) page.

To use a different module, pass '-m' for module name. For example, to use the 'ansible.builtin.shell' module:

```sh
ansible localhost -m ansible.builtin.shell -a 'echo $TERM'
# result:
#   localhost | CHANGED | rc=0 >>
#   xterm-256color

ansible localhost -m shell -a 'echo {{ ansible_version.full }}'
# result:
#   localhost | CHANGED | rc=0 >>
#   2.9.27
```

### Passing non-raw parameters to the module

Also use '-a' to provide options either through the k=v format (eg. -a 'opt1=val1 opt2=val2') or a JSON string starting with { and ending with } for more complex option structure (eg.  -a '{"opt1": "val1", "opt2": "val2"}'). You can learn more about [patterns](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html#intro-patterns) and [modules](https://docs.ansible.com/ansible/6/user_guide/modules.html#working-with-modules) on other pages.

```sh
# 'debug' module has a default value for msg
ansible localhost -m ansible.builtin.debug
# result:
#   localhost | SUCCESS => {
#       "msg": "Hello world!"
#   }

# print custom message
ansible localhost -m debug \
-a 'msg="Ansible version: {{ ansible_version.full }}"'
# result:
#   localhost | SUCCESS => {
#       "msg": "Ansible version: 2.9.27"
#   }

# copy a file
ansible localhost -m ansible.builtin.copy \
-a "src=/etc/hosts dest=./hosts"
# result:
#   localhost | CHANGED => {
#       "changed": true,
#       "checksum": "7335999eb54c15c67566186bdfc46f64e0d5a1aa",
#       "dest": "./hosts",
#       "gid": 1000,
#       "group": "vagrant",
#       "md5sum": "54fb6627dbaa37721048e4549db3224d",
#       "mode": "0664",
#       "owner": "vagrant",
#       "secontext": "unconfined_u:object_r:user_home_t:s0",
#       "size": 158,
#       "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1669947586.69-23147-117510746939328/source",
#       "state": "file",
#       "uid": 1000
#   }
```

## Running a simple playbook locally

**1 - Create the playbook**

```sh
cat <<EOT > ./playbook.yml
---
- hosts: all
  tasks:
    - debug:
        msg: Hello from Ansible
    - command: echo "This message is the output of a command"
      register: cmd_output
    - debug: msg="{{ cmd_output.stdout }}"
    - debug:
        msg: >
          {{ ansible_facts.date_time.date }} -
          {{ ansible_facts.date_time.time }}
          {{ ansible_facts.date_time.tz }}
          {{ ansible_facts.date_time.tz_offset }}
    - debug:
        msg:
          - "Python version: {{ ansible_facts.python_version }},"
          - "HOME: {{ ansible_facts.env.HOME }}"
EOT
```

NOTE: the 'hosts:' field is required.

**2 - Run it**

```sh
ansible-playbook -c local -i localhost, playbook.yml
```

Output:

```
PLAY [all] **************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [localhost]

TASK [debug] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "Hello from Ansible"
}

TASK [command] **********************************************************************************************************************
changed: [localhost]

TASK [debug] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "This message is the output of a command"
}

TASK [debug] ************************************************************************************************************************
ok: [localhost] => {
    "msg": "2022-12-02 - 05:56:13 UTC +0000\n"
}

TASK [debug] ************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "Python version: 2.7.5,",
        "HOME: /home/vagrant"
    ]
}

PLAY RECAP **************************************************************************************************************************
localhost                  : ok=6    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

See [Local playbooks](https://docs.ansible.com/ansible/5/user_guide/playbooks_delegation.html#local-playbooks) for details.

## Running an advanced playbook with other ansible artifacts

### Creating and running the playbook

In section I will be creating the following hierarchy.

```
provisioning
├── roles
│   ├── application           # this hierarchy represents the "application" role
│   │   └── tasks
│   │       └── main.yml
│   └── setup                 # this hierarchy represents the "setup" role
│       ├── handlers
│       │   └── main.yml
│       └── tasks
│           ├── docker.yml
│           └── main.yml
└── site.yml                  # main file
```

**1 - Create the playbook**

```sh
mkdir ./provisioning

cat <<EOT > ./provisioning/site.yml
---
- hosts: all
  tasks:
    - name: INFO - Python interpreter
      debug:
        msg: "{{ ansible_python_interpreter | default('/usr/bin/python') }}"

- name: Setup environment
  hosts: all
  roles:
    - setup

- name: Deploy app
  hosts: all
  roles:
    - application
EOT

mkdir -p ./provisioning/roles/setup/tasks/
mkdir -p ./provisioning/roles/setup/handlers/

cat <<EOT > ./provisioning/roles/setup/tasks/main.yml
---
- name: Update YUM cache
  yum:
    update_cache: yes
  tags:
    - system

- name: Install yum-utils
  yum:
    name: yum-utils
    state: present

- name: Setup docker
  include_tasks: docker.yml

- name: Remove unused packages
  yum:
    autoremove: yes
  tags:
    - system
EOT

cat <<EOT > ./provisioning/roles/setup/tasks/docker.yml
---
- name: Add YUM repo rocker-ce
  command: yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  args:
    creates: /etc/yum.repos.d/docker-ce.repo

- name: Update YUM cache
  yum:
    update_cache: yes
  tags:
    - system

- name: Install docker packages
  yum:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: present
  notify: Start docker

- command: uname -s
  register: uname_s_result

- command: uname -m
  register: uname_m_result

- name: Install docker-compose
  get_url:
    url: https://github.com/docker/compose/releases/download/1.29.2/docker-compose-{{ uname_s_result.stdout }}-{{ uname_m_result.stdout }}
    dest: /usr/local/bin/docker-compose
    mode: 0755

- file:
    src: /usr/local/bin/docker-compose
    dest: /usr/bin/docker-compose
    state: link
    mode: 0755
EOT

cat <<EOT > ./provisioning/roles/setup/handlers/main.yml
---
- name: Start docker
  systemd:
    name: docker
    state: started
    enabled: yes
EOT

mkdir -p ./provisioning/roles/application/tasks/

cat <<EOT > ./provisioning/roles/application/tasks/main.yml
---
- name: Run container for proxy server
  community.docker.docker_container:
    name: proxy
    image: nginx
    detach: true
    restart_policy: always
    ports:
        - 8080:80
EOT
```

**2 - Run it**

```sh
# validate
ansible-playbook -c local -i localhost, ./provisioning/site.yml --syntax-check

# run tasks with "system" tag
ansible-playbook -c local -i localhost, ./provisioning/site.yml --tags=system
# or run all tasks (it requires root privileges)
sudo su
export PATH=/home/vagrant/.local/bin:$PATH
ansible-playbook -c local -i localhost, ./provisioning/site.yml
# or simply
sudo env "PATH=/home/vagrant/.local/bin:$PATH" ansible-playbook \
  -c local -i localhost, \
  -e "ansible_python_interpreter=$(which python3.8)" \
  ${PWD}/provisioning/site.yml
```

NOTE: I needed '-e "ansible_python_interpreter=$(which python3.8)"' because otherwise ansible wont be able to find the Docker SDK for Python module. See [Ansible doesn't see an installed Python module when localhost is specified in the inventory](#ansible-doesnt-see-an-installed-python-module-when-localhost-is-specified-in-the-inventory) for details.

Output:

```
[vagrant@localhost ~]$ sudo env "PATH=/home/vagrant/.local/bin:$PATH" ansible-playbook   -c local -i localhost,   -e "ansible_python_interpreter=$(which python3.8)"   ${PWD}/provisioning/site.yml

PLAY [all] **************************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [localhost]

TASK [INFO - Python interpreter] ****************************************************************************************************
ok: [localhost] => {
    "msg": "/usr/local/bin/python3.8"
}

PLAY [Setup environment] ************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [localhost]

TASK [setup : Update YUM cache] *****************************************************************************************************
ok: [localhost]

TASK [setup : Install yum-utils] ****************************************************************************************************
ok: [localhost]

TASK [setup : Setup docker] *********************************************************************************************************
included: /home/vagrant/provisioning/roles/setup/tasks/docker.yml for localhost

TASK [setup : Add YUM repo rocker-ce] ***********************************************************************************************
ok: [localhost]

TASK [setup : Update YUM cache] *****************************************************************************************************
ok: [localhost]

TASK [setup : Install docker packages] **********************************************************************************************
ok: [localhost]

TASK [setup : command] **************************************************************************************************************
changed: [localhost]

TASK [setup : command] **************************************************************************************************************
changed: [localhost]

TASK [setup : Install docker-compose] ***********************************************************************************************
ok: [localhost]

TASK [setup : file] *****************************************************************************************************************
ok: [localhost]

TASK [setup : Remove unused packages] ***********************************************************************************************
ok: [localhost]

PLAY [Deploy app] *******************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [localhost]

TASK [application : Run container for proxy server] *********************************************************************************
changed: [localhost]

PLAY RECAP **************************************************************************************************************************
localhost                  : ok=16   changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0






[vagrant@localhost ~]$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
2fcb9ddcca80   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   proxy






[vagrant@localhost ~]$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Understanding the concepts behind some basic ansible artifacts

Ansible offers four distributed, re-usable artifacts: variables files, task files, playbooks, and roles.

See [Working with playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html) for more details.

#### Play

In summary, a play consists of:

```
- name: (optional, but recommended)
  hosts:
  tasks:
```

In a Playbook, anytime you come to the keywords 'hosts:' and 'tasks:', that indicates the start of a play (along with the optional 'name:' keyword).

#### Playbook

A playbook contains at least one play, and may contain variables, tasks, and other content. You can re-use tightly focused playbooks, but you can only re-use them statically, not dynamically.

#### The Role file structure

In Ansible, Roles is the mechanism that allows breaking a complex playbook into multiple files, that is reusable components. Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse and share them.

A role contains a set of related tasks, variables, defaults, handlers, and even modules or other plugins in a defined file-tree. Unlike variables files, task files, or playbooks, roles can be easily uploaded and shared even through Ansible Galaxy.

NOTE: When running a role, by default Ansible will look for a 'main.yml' file for relevant content (also 'main.yaml' and 'main') within the role. This is the entrypoint of the role, 'tasks/main.yml' is the main list of tasks that the role executes.

See [Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html), specially [Role directory structure](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure) and [Using roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#using-roles) for details.

#### Variables and variables file

Is it possible to create variables, either by defining them in a file (either in inventory, in playbooks, in reusable files, or in reusable files in roles), passing them at the command line, or registering the return value or values of a task as a new variable.

A variables file contains variables, that allows using them by a simple playbook or by a role.

See [Using Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html), specially [Where to set variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#where-to-set-variables) and [Organizing host and group variables](https://docs.ansible.com/ansible/latest//inventory_guide/intro_inventory.html#organizing-host-and-group-variables) sections for details.

#### Tasks and tasks file

A task is a call to an Ansible module with specific arguments.

A tasks file contains a reusable, ordered list of tasks to execute.

#### Tags

If you have a large playbook, it may be useful to run only specific parts of it instead of running the entire playbook. You can do this with Ansible tags. Using tags to execute or skip selected tasks is a two-step process:
- Add tags to your tasks, either individually or with tag inheritance from a block, play, role, or import.
- Select or skip tags when you run your playbook.

See [Tags](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html) for details.

#### Handlers and handlers file

Sometimes you want a task to run only when a change is made on a machine. For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged. Ansible uses handlers to address this use case. Handlers are tasks that only run when notified.

A handlers file contains a list of reusable handlers.

See [Handlers: running operations on change](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html) for details.

## Understanding variables

See [Working with playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html) for more details.

### Variables discovered from nodes (facts)

Facts represent discovered variables about a system. Ansible facts are data collected about the (target) systems on which Ansible takes actions. They are variables, but set by Ansible (in a way like 'system defined variables'). They are collected during 'Gathering Facts' stage of a playbook run, and it is controlled by the 'gather_facts' setting.

NOTE: You cannot access to ansible facts in 'ansible' ad hoc commands. To refer to ansible facts, you have to use a playbook.

The facts are gathered with 'setup' module. 'ansible-playbook' runs the setup module implicitly, so you will have access to all the facts as variables.

### Custom facts

The 'setup' module in Ansible automatically discovers a standard set of facts about each host. If you want to add custom values to your facts, you can
- write a custom facts module,
- set temporary facts with a 'ansible.builtin.set_fact' task,
- or provide permanent custom facts using the facts.d directory.

### Custom variables

You can define variables  (in a way like user defined variables) in a variety of places, such as
- in inventory,
- in playbooks,
- in reusable files,
- in roles,
- and at the command line.

Ansible loads every possible variable it finds, then chooses the variable to apply based on [variable precedence rules](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#ansible-variable-precedence).

See [Using variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) for details.

###  How to get nested values of data structures

To get values of registered variables (and facts) that are YAML or JSON data structures, you must use either bracket notation or dot notation. For example, to reference an IP address from your facts you can use one of the following notations:

```
# using the bracket notation
{{ ansible_facts["eth0"]["ipv4"]["address"] }}

# using the dot notation:
{{ ansible_facts.eth0.ipv4.address }}
```

See [Using Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) for details.

## Using collections

See [Using Ansible collections](https://docs.ansible.com/ansible/latest/collections_guide/index.html) and [Collection Index](https://docs.ansible.com/ansible/latest/collections/index.html) for details.

## Troubleshooting

### Verifying playbooks

It is possible to verify playbooks to catch syntax errors and other problems before you run them.

See [Verifying playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#verifying-playbooks) and [Tools for validating playbooks](https://docs.ansible.com/ansible/latest/community/other_tools_and_programs.html#tools-for-validating-playbooks) for details.

#### Syntax check

```sh
ansible-playbook -c local -i localhost, playbook.yml --syntax-check
```

#### List available tags

```sh
ansible-playbook -c local -i localhost, playbook.yml --list-tags
```

#### List tasks that would be executed

```sh
ansible-playbook -c local -i localhost, playbook.yml --list-tasks
```

#### Dry-run (aka. simulate)

This command don't make any changes; instead, try to predict some of the changes that may occur.

```sh
ansible-playbook -c local -i localhost, playbook.yml --check
```

Additionally, if a playbook tries to apply changes to files and templates, this command shows the differences in those files.

```sh
ansible-playbook -c local -i localhost, playbook.yml --diff
```

#### Linter

See [ansible-lint](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html#ansible-lint) for details.

### Get fact values locally

You can get ad hoc information about your system by running the following command:

```sh
# Get all facts
ansible localhost -m ansible.builtin.setup
```

Also you can apply a filter to get specific facts:

```sh
# Get environment variables
ansible localhost -m setup -a "filter=ansible_env"
```

For details, see:
- [Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)
- [Discovering variables: facts and magic variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html)
- [ansible.builtin.setup module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html)

### List installed collections

To list installed collections, run 'ansible-galaxy collection list'. This shows all of the installed collections found in the configured collections search paths.

NOTE: It will also show collections under development which contain a galaxy.yml file instead of a MANIFEST.json.

The path where the collections are located are displayed as well as version information. If no version information is available, a * is displayed for the version number.

```sh
ansible-galaxy collection list
# result:
#   # /home/vagrant/.local/lib/python3.8/site-packages/ansible_collections
#   Collection                    Version
#   ----------------------------- -------
#   amazon.aws                    3.5.0
#   ansible.netcommon             3.1.3
#   ansible.posix                 1.4.0
#   ansible.utils                 2.7.0
#   ansible.windows               1.12.0
#   arista.eos                    5.0.1
#   awx.awx                       21.8.0
#   azure.azcollection            1.14.0
#   check_point.mgmt              2.3.0
#   chocolatey.chocolatey         1.3.1
#   ...
```

### Ansible doesn't see an installed Python module when localhost is specified in the inventory

This issue is caused by a default behaviour of Ansible.

Even if you specify localhost in your inventory, Ansible will default to using '/usr/bin/python' for running the modules regardless of the 'connection: local' setting.

This in turn will cause problems if additional libraries were installed in a Python environment used to execute a playbook, but not for the '/usr/bin/python'.

The solution is to specify 'ansible_python_interpreter' setting for the localhost entry in the inventory.

References:
- [2.6.0 reports "Python modules \"botocore\" or \"boto3\" are missing, please install both" when localhost is specified in the inventory #42152](https://github.com/ansible/ansible/issues/42152)
- [Implicit 'localhost'](https://docs.ansible.com/ansible/2.6/inventory/implicit_localhost.html)
- [How do I handle not having a Python interpreter at /usr/bin/python on a remote machine?](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-handle-not-having-a-python-interpreter-at-usr-bin-python-on-a-remote-machine)

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

  # NFS requires a host-only network to be created.
  # We need to add a host-only network to the machine (with either DHCP or a static IP) for NFS to work.
  config.vm.network "private_network", ip: "192.168.33.11"

  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.synced_folder "./shared", "/var/shared", disabled: true,
  type:"nfs",
  create: true,
  # group:'nginx', owner:'magento', # no supported by NFS
  mount_options: %w{rw,async,fsc,nolock,vers=3,udp,rsize=32768,wsize=32768,hard,noatime,actimeo=2}
end
```

**provision.sh**

```sh
set -x #echo on

# Update the system
sudo yum -y update

# Add the epel repository
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Install wget
sudo yum -y install wget

# Install dev tools to build python 8
sudo yum -y groupinstall "Development Tools"
sudo yum -y install openssl-devel bzip2-devel libffi-devel xz-devel

# Download and extract python 3.8
wget https://www.python.org/ftp/python/3.8.15/Python-3.8.15.tgz
tar xvf Python-3.8.15.tgz
cd Python-3.8.15/
# Configure and build
./configure --enable-optimizations
# Compile and install
sudo make altinstall
# Delete downloaded stuff
cd ..
sudo rm -rf Python-3.8.15*

# Update pip3
pip3.8 install --upgrade pip

# Install ansible
pip3.8 install ansible

# Upgrade
pip3.8 install --upgrade ansible

# Modules and plugins in community.docker require the Docker SDK for Python.
# The SDK needs to be installed on the machines where the modules and plugins are executed,
# and for the Python version(s) with which the modules and plugins are executed.
pip3.8 install docker

# Install ansible and docker SDK for python, for root user
sudo env "PATH=/home/vagrant/.local/bin:$PATH" pip3.8 install ansible
sudo env "PATH=/home/vagrant/.local/bin:$PATH" pip3.8 install --upgrade ansible
sudo env "PATH=/home/vagrant/.local/bin:$PATH" pip3.8 install docker

# Include 'ansible' bin dir in PATH
echo 'export PATH=~/.local/bin:$PATH' >> ~/.bashrc
```

