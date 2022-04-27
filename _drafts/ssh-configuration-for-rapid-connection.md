
# Configuring SSH for rapid connection

In the following examples I assume my remote server is '192.168.100.7', port '22,' and user 'root', you will have to change these values accordingly for your use case.

## Configure an alias

### Using ~/.ssh/config

1. Create the ~/.ssh directory if necessary:

```sh
mkdir -p ~/.ssh && chmod 700 ~/.ssh
```

2. Create or edit the SSH configuration file:

```sh
vi ~/.ssh/config
```

3. Add the remote host details like below for a long command like `ssh -p 22 root@192.168.100.7`:

```
Host rpi4
    HostName 192.168.100.7
    User root
```

4. Make the file readable and writable only by the user and not accessible by others:

chmod 600 ~/.ssh/config

5. Now you can SSH into the the remote host using the alias Host name:

```sh
ssh rpi4
```

### Using bash aliases

1. Open ~/.bashrc:

```sh
vi ~/.bashrc
```

2. Add aliases for the SSH connection like below:

```sh
alias ssh-rpi4='ssh -p 22 root@192.168.100.7'
```

3. Apply the changes to current session using the 'source' command:

```sh
source ~/.bashrc
```

4. Now you can use the alias name like below:

```sh
ssh-rpi4
```

## Key-based authentication - Passwordless SSH login

1. Generate a SSH key pair:

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
# or
# ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"

# Enter file in which to save the key (/home/yourusername/.ssh/id_ed25519):
## WRITE FOR EXAMPLE /home/yourusername/.ssh/id_ed25519_rpi4
# Enter passphrase (empty for no passphrase):
## PRESS ENTER
# Enter same passphrase again:
## PRESS ENTER
# Your identification has been saved in /home/yourusername/.ssh/id_ed25519_rpi4
# Your public key has been saved in /home/yourusername/.ssh/id_ed25519_rpi4.pub
```

2. Configure the connection to use the correct key. Run `vi ~/.ssh/config` and add the 'IdentityFile' configuration for the remote host like below:

```
Host rpi4
    HostName 192.168.100.7
    User root
    IdentityFile ~/.ssh/id_ed25519_rpi4
```

Optionally, you could add the key to the current session:

```sh
eval `ssh-agent`
ssh-add ~/.ssh/id_ed25519_rpi4
```

3. Copy the public key to the remote server you want to manage:

```sh
ssh-copy-id -i /home/yourusername/.ssh/id_ed25519_rpi4.pub root@192.168.100.7
# or
# cat ~/.ssh/id_rsa.pub | \
# ssh rpi4 \
#   "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Password:
# PUT THE PASSWORD OF THE REMOTE USER
```

4. Now you should be aable to run for example `ssh rpi4` without putting any password.

## Disable SSH Password Authentication

1. Log into the remote server with SSH keys, either as a user with sudo privileges or root:

```sh
ssh root@192.168.100.7
```

2. Open the SSH configuration file /etc/ssh/sshd_config:

```sh
vi /etc/ssh/sshd_config
```

3. Search for the following directives and modify as it follows:

```
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```

1. Save the file and restart the SSH service:

```sh
# On Ubuntu or Debian servers, run the following command:
sudo systemctl restart ssh

# On CentOS or Fedora servers, run the following command:
sudo systemctl restart sshd
```
