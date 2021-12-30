---
title: "Managing SSH Keys for different github accounts on the same machine"
tags: ssh git github gitlab
---

# Managing SSH Keys for different github accounts on the same machine

NOTE: I exaplain how to make it with github, but you can apply the following steps also when configuring SSH keys for other services similar to github like gitlab, bitbucket, etc.

## Generate a SSH key

In the following command, replace 'YOURACCOUNT' with the name of your account in github, and 'your_email@example.com' with the email associated to that account.

```sh
ssh-keygen -t ed25519 -N "" -f "$HOME/.ssh/id_ed25519_YOURACCOUNT" -C "your_email@example.com"

# If you are using a legacy system that doesn't support the Ed25519 algorithm, use:
# ssh-keygen -t rsa -b 4096 -N "" -f "$HOME/.ssh/id_rsa_YOURACCOUNT" -C "your_email@example.com"
```

## Configure SSH adding a Host for each SSH key you created

Open the config file with `vi ~/.ssh/config` and configure it with your SSH keys and custom connections.

```sh
# YOURACCOUNT account
Host github.com-YOURACCOUNT
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_ed25519_YOURACCOUNT

# YOURACCOUNT2 account
Host github.com-YOURACCOUNT2
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_ed25519_YOURACCOUNT2
```

You could also add configurations for other connections into this file, for example you could add a custom name for a connection to a remote machine using the following template:

```
# Replace 'My_Remote_Machine My_Remote_Machine2 My_Remote_Machin3' with a custom name or multiple custom names separated by spaces
# Replace 'Real_IP_Address_Or_Hostname' with the real IP/hostname SSH should use when connecting
# Replace 'Username' with the user account SSH should use when connecting

Host My_Remote_Machine My_Remote_Machine2 My_Remote_Machin3
    HostName Real_IP_Address_Or_Hostname
    User Username
```

Then you can type `ssh My_Remote_Machine` - or `ssh My_Remote_Machine2` or `ssh My_Remote_Machine3` - on the command line and ssh will automatically do `ssh Username@Real_IP_Address_Or_Hostname` for you.

## Start the ssh-agent and add your SSH private keys.

```sh
# Start the ssh-agent in the background
eval "$(ssh-agent -s)"

# (Optional) Delete all cached keys
ssh-add -D

# Add your SSH private keys to the ssh-agent
ssh-add ~/.ssh/id_ed25519_YOURACCOUNT
ssh-add ~/.ssh/id_ed25519_YOURACCOUNT2

# Check the saved keys
ssh-add -l
```

## Associate your SSH public keys to github

Run `cat ~/.ssh/id_ed25519_YOURACCOUNT.pub` and add the string as a recognized SSH key in your github acconut.

## Clone the repository and modify the configuration locally

### Clone the repository

Clone the repository copying it from github but changing 'github.com' after the @ symbol and using one of the SSH hosts you defined above:

```sh
# SSH link format for git repositories
# git@SSHHOST:REPO_OWNER_ACCOUNT/REPO_NAME.git
# e.g.
git clone git@github.com-YOURACCOUNT:aldoestebanpaz/aldoestebanpaz.github.io.git
```

Alternatively, if you cloned the repo without changing the host, you can open the config file of the cloned repo doing `vi .git/config` and changing the 'url' field defined in the section '[remote "origin"]':

```
[remote "origin"]
        url = git@github.com-YOURACCOUNT:aldoestebanpaz/aldoestebanpaz.github.io.git
```

### Configure user.name and user.email locally

You need to configure user.name and user.email with the one you have for your github account for that repository because otherwise you will be using the same global user.name and user.email on all repositories of all github accounts you have configured.

```sh
cd aldoestebanpaz.github.io
git config user.name "YOURACCOUNT"
git config user.email "your_email@example.com"
```
