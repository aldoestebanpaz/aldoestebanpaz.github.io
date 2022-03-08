Searching a package

NOTE: Remember to update the index (the list of available packages from the repositories) ocasionally before to run apt-cache or apt-get.
sudo apt-get update

# Search in package index
apt-cache search <name>

# Search in list of installed packages
dpkg -l | grep <name>


Show information about a package

# Show information about the package
apt-cache show <package>

# Show version and architecture
dpkg -l <package>

# Check required disk space
sudo apt-get install --assume-no <package>

# Show more details about the package (includes depends and rdepends)
apt-cache showpkg <package>

# List package dependencies and recomendations
apt-cache depends <package>

# List reverse dependencies
apt-cache rdepends <package>

# List available versions of a package
apt-cache madison <package>


Installation

# Download package source
apt-get source <package>

# Simulate install
No action; perform a simulation of events that would occur based on the current system state but do not actually change the system.
sudo apt-get install -s <package>

# Install or Upgrade a package
If package is already installed it will try to update to latest version
sudo apt-get install <package>

# Upgrade all installed packages, if possible
sudo apt-get update && sudo apt-get upgrade


Package configuration with debconf

# Show available configuration scripts
Roughly 5% of Debian packages use the debconf database and custom scripts using that data to generate configuration files. These scripts are located in /var/lib/dpkg/info/*.config.
ls -l /var/lib/dpkg/info/*.config

# Reconfigure an already installed package
sudo dpkg-reconfigure <package>


For a package already installed

# Only upgrade (no install)
Will install upgrades for already installed packages only and ignore requests to install new packages
sudo apt-get --only-upgrade install <package>

#  Search the package that provides the sqlite3 module for pkg-config
dpkg -S /usr/lib/x86_64-linux-gnu/pkgconfig/sqlite3.pc
dpkg -S sqlite3.pc

# List the files inside the intalled package
dpkg -L libsqlite3-dev:amd64
dpkg -L libsqlite3-dev

# Get information about the installed package
dpkg -s libsqlite3-dev

# List installed dependencies
apt-rdepends --state-follow=Installed --state-show=Installed acpid

# List installed reverse dependencies
apt-rdepends --state-follow=Installed --state-show=Installed --reverse acpid



For a package non installed yet


# Install 'apt-file' if you dont have it already installed.
sudo apt-get install apt-file

NOTE: Remember to run `sudo apt-file update` before using the apt-file command whenever possible.

#  Search the package that provides the glib-2.0 module for pkg-config
apt-file search glib-2.0.pc

# List the files inside a remote non-intalled package
apt-file list libglib2.0-dev

# List the files inside a .deb files
dpkg-deb -c ./<package>.deb
