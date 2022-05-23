
## Common commands for Portage

### Install a package

Example:

```sh
sudo emerge app-editors/vim
# or
sudo emerge -atv app-editors/vim
```

The options (-atv) are short options for --ask, --tree, and --verbose. They trigger emerge to ask before proceeding, display the dependency tree of packages to be installed, and to be verbose with its output.

### Searching a package

Example listing packages containing the 'vim':

```sh
emerge --quit --search vim
# emerge --search vim throws additional info about the package but it gets difficult for reading.
```

### Fixing the 'masked by: missing keyword' message

Some packages have not been marked as available for installation on a specific architectures like 'arm'. But still a lots of packages may compile on an architecture even if there is no ~arch keyword added to them.

Example:

```
!!! All ebuilds that could satisfy "app-portage/pfl" have been masked.
!!! One of the following masked packages is required to complete your request:
- app-portage/pfl-3.1::gentoo (masked by: missing keyword)
```

then the following fixes it:

```sh
sudo vim /etc/portage/package.accept_keywords/app-portage-pfl
```

content:

```
# Recommended.
# Get the latest version but not any live ebuild (git/svn etc)
<app-portage/pfl-9999 **

# Some other examples follow.

# To accept the latest version.
# This will pull in live git/svn ebuild if there is one which may not
# be what was intended. Live ebuilds also need entries for package.unmask.
app-portage/pfl **

# Or like this, pinning a particular version.
=app-portage/pfl-3.1 **
```

### Unmask automatically with 'emerge --autounmask-write'

Running the emerge command with '--autounmask-write' automatically queues the required text file alteration:

Example:

```sh
sudo emerge --ask --autounmask-write =xorg-server-1.11.99.2
```

As with all configuration files, the final update is dispatched by running `dispatch-conf` (or alternatively `etc-update`):

```sh
dispatch-conf
```

Examine the diff output of the configuration files, press 'q' to exit the pager (if no commands appear at the end), and then 'u' for use-new to accept the alterations. Press 'z' to zap (disregard) the changes.

Finally, re-run emerge one last time:

```sh
emerge --ask =xorg-server-1.11.99.2
```

See also [Knowledge Base:Autounmask-write](https://wiki.gentoo.org/wiki/Knowledge_Base:Autounmask-write) for more information.

### Unmask manually

Example using flat file:

```sh
# Create the /etc/portage directory if it does not yet exist:
sudo mkdir -p /etc/portage
# Add the required line to a (new) package.unmask file:
sudo echo "=x11-base/xorg-server-1.11.99.2" >> /etc/portage/package.unmask
```

Example using directories:

```sh
# Create the /etc/portage/package.unmask directory if it does not yet exist:
mkdir -p /etc/portage/package.unmask

# Create a file (or files) for the unmask operations to perform. For instance, to unmask the xorg-server-1.11.99.2 package as mentioned earlier, run:
sudo echo "=x11-base/xorg-server-1.11.99.2" > /etc/portage/package.unmask/xorg-server
```

Most masked packages will not accept generic keywords, so specific keywords may need to be granted for the masked package.

### Fixing the 'The following USE changes are necessary to proceed' meesage

If you see the line "The following USE changes are necessary to proceed" when trying to install a package, then you will have to add the green lines below this message (only the 3 green ones are important but you can copy the comments with it so that you know why they are here) to the `/etc/portage/package.use` file or directory. If there is no `/etc/portage/package.use`, feel free to create it.

Example:

```
The following USE changes are necessary to proceed:
 (see "package.use" in the portage(5) man page for more details)
# required by app-portage/pfl-3.1::gentoo[network-cron]
# required by app-portage/pfl (argument)
=sys-apps/util-linux-2.35.2 caps
```

then the following fixes it using falt file:

```sh
sudo echo '=sys-apps/util-linux-2.35.2 caps' >> /etc/portage/package.use
```

or the following using directories:

```sh
sudo vim /etc/portage/package.use/app-portage-pfl
```

content:

```
=sys-apps/util-linux-2.35.2 caps
```

## Troubleshooting

### List ebuild repositories

The list of active ebuild repositories can be obtained through the output of one of the following commands:

```sh
emerge --info
# or
portageq repos_config /
```

### Get path to the ebuild file

The following command shows the path to the ebuild that would be used by Portage with the current configuration.

```sh
equery which <package>
```

### List the mirrors for source package download

These mirrors will be used to download a source package.

The GENTOO_MIRRORS variable is set in `/usr/share/portage/config/make.globals`, but it can be overwritten with an entry in `/etc/portage/make.conf`.

```sh
cat /usr/share/portage/config/make.globals | grep GENTOO_MIRRORS
# prints, for example:
#   GENTOO_MIRRORS="http://distfiles.gentoo.org"
cat /etc/portage/make.conf | grep GENTOO_MIRRORS
# prints, for example:
#   GENTOO_MIRRORS="http://gentoo.osuosl.org/ http://trumpetti.atm.tut.fi/gentoo/ http://distfiles.gentoo.org"
```

### Get the hosts of binary packages

When using a binary package host, clients need to have the 'PORTAGE_BINHOST' variable set in `/etc/portage/make.conf` or the 'sync-uri' variable in `/etc/portage/binrepos.conf`. Otherwise, the client will not know where the binary packages are stored which results in Portage being unable to retrieve them.

The 'PORTAGE_BINHOST' variable uses a space-separated list of URIs. This allows administrators to use several binary package servers simultaneously. The URI must always point to the directory in which the 'Packages' file resides.

```sh
cat /etc/portage/make.conf | grep PORTAGE_BINHOST
# prints, for example:
#   PORTAGE_BINHOST="https://isshoni.org/pi64pie"
cat /etc/portage/binrepos.conf | grep sync-uri
# pront, for example:
#   sync-uri = https://example.com/binhost
```