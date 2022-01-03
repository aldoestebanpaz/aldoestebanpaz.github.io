---
title: Version Managers - Installation and Condiguration
tags: multilanguage
---

# Version Managers - Installation and Condiguration

## Node.js

### For Linux

nvm (Node Version Manager) is a version manager for node.js that allows you to quickly install and use different versions of node via the command line.

nvm is designed to be installed per-user, and invoked per-shell. nvm works on any POSIX-compliant shell (sh, dash, ksh, zsh, bash), in particular on these platforms: unix, macOS, and windows WSL.

1. Download the installation script and runs it:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# or
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```
2. Close your current terminal, open a new terminal, and run `command -v nvm` to check if the shell can find nvm. Alternatively, you can run `bash: source ~/.bashrc` for the different shells on the command line.

**What the script did?**

The script clones the nvm repository to ~/.nvm, and attempts to add the following lines (a.k.a. source lines) to the correct profile file (~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc) automatically:

```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

**Installing and using an node version**

1. List available versions to install: `nvm ls-remote --lts`.
2. Install the selected version: `nvm install v16.13.0`.
3. (optional) List installed versions: `nvm ls`.
4. Switch to the installed version: `nvm use v16.13.0`.

**Configuring an specific node version in a project directory**

1. Create the .nvmrc file with the version: `echo "5.9" > .nvmrc`.
2. Move to the project directory and run: `nvm use`. This should create an .nvmrc file with a <version> value (as described by nvm --help) followed by the required newline.

NOTE: No trailing spaces are allowed, and the trailing newline is required.

**Useful commands**

- Attempt to upgrade to the latest working npm on the current node version: `nvm install-latest-npm`.
- Display path to installed node version (uses .nvmrc if available and version is omitted): `nvm which [current | <version>]`.


### For Windows

nvm also support Windows in some cases. It should work through WSL (Windows Subsystem for Linux) depending on the version of WSL. It should also work with GitBash (MSYS) or Cygwin. Otherwise, for Windows, a few alternatives exist like nvm-windows.

TODO

Reference:
https://github.com/nvm-sh/nvm
https://github.com/coreybutler/nvm-windows

## Ruby

TODO

## Python

TODO

