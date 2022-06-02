
<!-- TOC -->

- [Finding packages and source code on Linux distros](#finding-packages-and-source-code-on-linux-distros)
  - [Finding the process that a window (GUI) belongs to](#finding-the-process-that-a-window-gui-belongs-to)
    - [Output the PID](#output-the-pid)
    - [Output PPID, PID and command](#output-ppid-pid-and-command)
  - [dpkg and apt based distros like Debian and Ubuntu](#dpkg-and-apt-based-distros-like-debian-and-ubuntu)
    - [Binary package vs Source package](#binary-package-vs-source-package)
      - [Binary package](#binary-package)
      - [Source package](#source-package)
    - [Finding the installed package that a file belongs to](#finding-the-installed-package-that-a-file-belongs-to)
    - [Listing files inside the installed package](#listing-files-inside-the-installed-package)
    - [Extracting information from the package](#extracting-information-from-the-package)
    - [Finding the binary package](#finding-the-binary-package)
      - [In Debian](#in-debian)
      - [In Ubuntu](#in-ubuntu)
    - [Finding the source spackage in Debian](#finding-the-source-spackage-in-debian)
      - [In Debian](#in-debian-1)
      - [In Ubuntu](#in-ubuntu-1)
    - [Finding the repository with files for packaging](#finding-the-repository-with-files-for-packaging)
      - [In Debian](#in-debian-2)
      - [In Ubuntu](#in-ubuntu-2)
    - [Finding the upstream (original) repository](#finding-the-upstream-original-repository)
      - [In Debian](#in-debian-3)
      - [In Ubuntu](#in-ubuntu-3)
    - [How to read the changelog of a package](#how-to-read-the-changelog-of-a-package)
    - [How to download a binary package](#how-to-download-a-binary-package)
    - [How to download a source package](#how-to-download-a-source-package)
  - [portage based distros like Gentoo](#portage-based-distros-like-gentoo)
    - [Ebuilds and binary packages](#ebuilds-and-binary-packages)
      - [Ebuilds (scripts source packages)](#ebuilds-scripts-source-packages)
      - [Binary packages](#binary-packages)
    - [Finding the installed package that a file belongs to](#finding-the-installed-package-that-a-file-belongs-to-1)
    - [Listing files inside the installed package](#listing-files-inside-the-installed-package-1)
    - [Extracting information from the package](#extracting-information-from-the-package-1)
    - [Finding the package](#finding-the-package)
      - [On the web page](#on-the-web-page)
    - [How to download a binary package](#how-to-download-a-binary-package-1)
    - [How to download a source package](#how-to-download-a-source-package-1)
      - [The exact location of the source package in a mirror](#the-exact-location-of-the-source-package-in-a-mirror)

<!-- /TOC -->

# Finding packages and source code on Linux distros

In the following examples I show how to proceed for finding the package and source code project behind the `lxqt-admin-user` binary.

## Finding the process that a window (GUI) belongs to

It is possible to discover which process a window belongs to by using the `xprop` command line tool (which is part of the Debian package `x11-utils`).

When you run tools like `xprop` or `xwininfo` you have to click on the target window. If the desired window is not visible, use Alt+Tab to bring it to the foreground.

### Output the PID

Run either:

```sh
xprop _NET_WM_PID
# prints:
#   _NET_WM_PID(CARDINAL) = 22044
```

or:

```sh
xprop _NET_WM_PID | sed 's/_NET_WM_PID(CARDINAL) = //'
# prints:
#   22044
```

### Output PPID, PID and command

Run the following to output the parent process ID (PPID), process ID (PID) and the complete command used for executing a process:

```sh
ps ww -o ppid=,pid=,cmd= -q `xprop _NET_WM_PID | sed 's/_NET_WM_PID(CARDINAL) = //'`
# prints:
#   1   22044 /usr/bin/lxqt-admin-user
```

## dpkg and apt based distros like Debian and Ubuntu

### Binary package vs Source package

#### Binary package

A Binary package is a file that ends in .deb and contains software for your Debian system. A Debian package is smart enough to know how to add itself to your system, remove itself, and even configure itself to your needs.

`ar t <debfile>` displays the list of files contained in a .deb archive.

A Debian package (.deb) is an `ar` archive that is comprised of three files:

- `debian-binary` - This is a text file which simply indicates the version of the .deb file package format version. In Debian Buster it is still version 2.0.
- `control.tar.xz`- This archive file contains all of the available meta-information, like the name and version of the package as well as some scripts to run before, during or after (un-)installation of it. Some of the meta-information allows package management tools to determine if it is possible to install or uninstall it, for example according to the list of packages already on the machine, and if files shipped have been modified locally.
- `data.tar.xz`, `data.tar.bz2`, `data.tar.gz` - This archive contains all of the files to be extracted from the package; this is where the executable files, libraries, documentation, etc., are all stored. Packages may use different compression formats, in which case the file will be named differently for xz, bzip2 or gzip.

More info: https://debian-handbook.info/browse/stable/packaging-system.html

#### Source package

Source packages provide you with all of the necessary files to compile or otherwise, build the desired piece of software.

A Debian source package in the "3.0 (quilt)" format consists of the following files:

- `package_version.orig.tar.xz` - The upstream tarball. This file could be a copy or symlink of `package-version.tar.gz`.
- `package_version-revision.debian.tar.xz` - The debianization patch - Historically it was known as the '.diff.gz' file in 1.0 format, it is tarball of `package-version/debian/*`, the Debian package specification files. These files are added to the upstream source under the `package-version/debian/` directory.
- `package_version-revision.dsc` - The Debian source control file. The file is generated by `dpkg-source` when it builds the source archive from other files in the source package. When unpacking, it is checked against the files and directories in the other parts of the source package. More info: https://www.debian.org/doc/debian-policy/ch-controlfields.html.
- `package_version.orig.tar.xz.asc` - A PGP signature file used to verify the integrity of the upstream tarball.

In our example for `lxqt-admin` in ubuntu hirsute, the files are the following:

```
lxqt-admin-0.16.0
lxqt-admin_0.16.0-1ubuntu1.debian.tar.xz
lxqt-admin_0.16.0-1ubuntu1.dsc
lxqt-admin_0.16.0.orig.tar.xz
lxqt-admin_0.16.0.orig.tar.xz.asc
```

More info: https://www.debian.org/doc/manuals/debmake-doc/ch05.en.html

### Finding the installed package that a file belongs to

Run either:

```sh
dpkg -S /usr/bin/lxqt-admin-user
# prints:
#   lxqt-admin: /usr/bin/lxqt-admin-user
```

or, if it is a binary:

```sh
dpkg -S `which lxqt-admin-user`
# prints:
#   lxqt-admin: /usr/bin/lxqt-admin-user
```

### Listing files inside the installed package

Run:

```sh
dpkg -L lxqt-admin
# prints:
#   /.
#   /usr
#   /usr/bin
#   /usr/bin/lxqt-admin-time
#   /usr/bin/lxqt-admin-user
#   /usr/bin/lxqt-admin-user-helper
#   /usr/share
#   /usr/share/applications
#   /usr/share/applications/lxqt-admin-time.desktop
#   /usr/share/applications/lxqt-admin-user.desktop
#   /usr/share/doc
#   /usr/share/doc/lxqt-admin
#   /usr/share/doc/lxqt-admin/AUTHORS
#   /usr/share/doc/lxqt-admin/README.md.gz
#   /usr/share/doc/lxqt-admin/changelog.Debian.gz
#   /usr/share/doc/lxqt-admin/copyright
#   /usr/share/lintian
#   /usr/share/lintian/overrides
#   /usr/share/lintian/overrides/lxqt-admin
#   /usr/share/polkit-1
#   /usr/share/polkit-1/actions
#   /usr/share/polkit-1/actions/org.lxqt.lxqt-admin-user.policy
```

### Extracting information from the package

You can run the following commands for extracting information about the installed package:
- `dpkg -s lxqt-admin` - shows description, dependencies, and project homepage.
- `apt-cache show lxqt-admin` - shows description, dependencies, and project homepage as well.
- `apt-cache showpkg lxqt-admin` - shows versions, dependencies, and reverse dependencies.
- `/usr/share/doc/lxqt-admin/copyright` - sometimes shows the link to the original page/repository of the project.
- `zless /usr/share/doc/lxqt-admin/README.md.gz` - the README file can contain any kind of information, it is usually compressed with gzip and that is why we use `zless`.

### Finding the binary package

Note "binary package" and "package" are synonyms.

#### In Debian

You can either:

- Generate an URL using `https://packages.debian.org/{PAKAGE_NAME}` and your package name. For example, `https://packages.debian.org/lxqt-admin`.
- Generate an URL using `https://packages.debian.org/search?keywords={PAKAGE_NAME}` and your package name. For example, `https://packages.debian.org/search?keywords=lxqt-admin`.
- Going to `https://packages.debian.org/` and searching the package manually.

The page shows multiple options all related to an specific debian release (aka suite) codename like strech, buster, bullseye, bookworm and sid (the development distribution of Debian). You can get more information about Debian suites in [https://wiki.debian.org/DebianReleases](https://wiki.debian.org/DebianReleases) and also you can see what suite your distro is using by running `cat /etc/debian_version`.

The option "sid" ([Debian Unestable](https://wiki.debian.org/DebianUnstable)) is probably the best option. It is not strictly a release, but rather a "rolling development" version of the Debian distribution containing the latest packages that have been introduced into Debian.

For this example I can see the most up to date information about a package going to `https://packages.debian.org/sid/lxqt-admin`.

#### In Ubuntu

You can either:

- Generate an URL using `https://packages.ubuntu.com/{PAKAGE_NAME}` and your package name. For example, `https://packages.ubuntu.com/lxqt-admin`.
- Generate an URL using `https://packages.ubuntu.com/search?keywords={PAKAGE_NAME}` and your package name. For example, `https://packages.ubuntu.com/search?keywords=lxqt-admin`.
- Going to `https://packages.ubuntu.com/` and searching the package manually.

The page shows multiple options all related to an specific ubuntu release codename like hirsute (Ubuntu 21.04), bionic (Ubuntu 18.04) and focal (Ubuntu 20.04). You can get more information about Ubuntu releases in [https://wiki.ubuntu.com/Releases](https://wiki.ubuntu.com/Releases) and [https://ubuntu.com/about/release-cycle](https://ubuntu.com/about/release-cycle), and also you can see what release your distro is using by running `lsb_release -a`.

For this example I can see the information about a package for hirsute going to `https://packages.ubuntu.com/hirsute/lxqt-admin`. This page shows the title "Package: lxqt-admin (0.16.0-1ubuntu1) [universe]" that means that this package is located in the repository called "universe".

**What about an unestable release in Ubuntu?**

Source: https://askubuntu.com/questions/1341024/is-there-ubuntu-unstable-development-like-debian-sid-unstable

There is no "rolling development" version for Ubuntu like Sid on Debian. In other words, there's no singular name like sid or Unstable in Debian to constantly track 'development'.

You will need to keep upgrading/installing the in-development releases during the dev cycles to get the 'latest development' track. The current in-development release appears in the 'Future' section in [https://wiki.ubuntu.com/Releases](https://wiki.ubuntu.com/Releases).

Also there is no 'magical' way to continue to track the development or unstable branches after a release is complete - except to 'upgrade' to the development release via `do-release-upgrade -d` on that system. However, that isn't guaranteed to work, so you'd have to do a reinstall with the ISOs for that development release to guarantee you're on that devel track, and repeat the same behavior every time a new release is being developed.

### Finding the source spackage in Debian

A "binary package" (or just "package") is different than a "source package". A source package could be the source of more than one package, in other words one or more binary packages can be derived from a single source package. For example, the page for the source package "shadow" (the shadow tools aka. shadow-utils, https://packages.debian.org/source/sid/shadow) shows that binary packages "libsubid-dev", "libsubid4", "login", "passwd" and "uidmap" are built from it.

In our example the name of both the package and the source package are the same. But from the source package lxqt-admin derives two binary packages, lxqt-admin and lxqt-admin-l10n.

#### In Debian

You can get the link to the source package from a specific package page in packages.debian.org. For example, if you go to `https://packages.debian.org/sid/lxqt-admin` you will see a link to the source package page in the top-left side of the page: `https://packages.debian.org/source/sid/lxqt-admin`.

Other alternative is opening the "Developer Information" link that appears in the same page in the right side. In this example it is `https://tracker.debian.org/pkg/lxqt-admin`. This page shows the link to the source package too, `https://packages.debian.org/src:lxqt-admin` in this case.

`https://packages.debian.org/src:lxqt-admin` shows the options for the different debian releases, while `https://packages.debian.org/source/sid/lxqt-admin` is the source package specific for the sid (unestable) suite.

#### In Ubuntu

You can get the link to the source package from a specific package page in packages.ubuntu.com. For example, if you go to `https://packages.ubuntu.com/hirsute/lxqt-admin` you will see a link to the source package page in the top-left side of the page: `https://packages.ubuntu.com/source/hirsute/lxqt-admin`.

A second alternative is looking the "Download Source Package" link that appears in the same page in the right side.

Another alternative is using the `https://packages.ubuntu.com/src:name` template, for example `https://packages.ubuntu.com/src:lxqt-admin`. This page shows the options for the different ubuntu releases.

### Finding the repository with files for packaging

#### In Debian

You can get the link:
- Opening the source package page `https://packages.debian.org/source/sid/lxqt-admin` and clicking the "Debian Source Repository" link.
- Opening the Developer Information page `https://tracker.debian.org/pkg/lxqt-admin` and clicking the VCS's Browse link.

The link in this example is `https://salsa.debian.org/lxqt-team/lxqt-admin`.

#### In Ubuntu

You can get the link in the source package page:
- In the "Debian Source Repository" link at the right side.
- In the "Debian Package Source Repository (Browsable)" link at the buttom of the page.

The link in this example is `https://phab.lubuntu.me/source/lxqt-admin/`.

### Finding the upstream (original) repository

#### In Debian

You can get the link:
- Opening the binary package page `https://packages.debian.org/sid/lxqt-admin` and clicking the "Homepage" link.
- Opening the source package page `https://packages.debian.org/source/sid/lxqt-admin` and clicking the "Homepage" link.
- Opening the Developer Information page `https://tracker.debian.org/pkg/lxqt-admin` and clicking the "homepage" link.
- Following one of the options listed in the "Extracting information from the package" section above.

The link in this example is `https://github.com/lxqt/lxqt-admin`.

#### In Ubuntu

You can get the link in the source package page. You can find it in the "Homepage" link at the right side.

### How to read the changelog of a package

```sh
apt changelog lxqt-config
```

### How to download a binary package

```sh
# Download the .deb archive file
apt-get download lxqt-admin
# List the content of the binary package
lesspipe ../hello_2.12-0ubuntu1_amd64.deb
# Extract the files from the archive
mkdir ./deb_bin_files && ar vx --output ./deb_bin_files lxqt-admin_0.16.0-1ubuntu1_amd64.deb
# Extract the .xz (with -J) archive with installable files
cd deb_bin_files
mkdir ./data && tar -xJvf data.tar.xz -C ./data
```

### How to download a source package

1. Add or uncomment the `deb-src` entry in your `/etc/apt/sources.list` file. In my case I'm using Ubuntu 21.04 (hirsute) here:

```sh
# sudo vi /etc/apt/sources.list

# Main (Canonical-supported free and open-source software)
# and Restricted (Proprietary drivers for devices) repositories.
deb-src http://archive.ubuntu.com/ubuntu/ hirsute main restricted
# Universe (Community-maintained free and open-source software) repository.
deb-src http://archive.ubuntu.com/ubuntu/ hirsute universe
```

2. Retrieve the updated package lists:

```sh
sudo apt-get update
```

3. Download the source package:

```sh
# Download all the files and unpack the tarballs
apt-get source lxqt-admin

# Download the upstream's tarball only
apt-get source --tar-only lxqt-admin

# Download the debianization patch (.debian.tar.xz) only
apt-get source --diff-only lxqt-admin

# Download the Debian source control file (.dsc) only
apt-get source --dsc-only lxqt-admin
```

## portage based distros like Gentoo

### Ebuilds and binary packages

#### Ebuilds (scripts source packages)

TODO

#### Binary packages

TODO

### Finding the installed package that a file belongs to

The 'b' or 'belongs' module of the equery command lists the package that owns a file: `equery belongs|b [OPTIONS] <file>`.

Normally, only one package will own the file. If multiple packages own the same file it should be reported.

Example:

```sh
equery b `which emerge`
# prints:
#   sys-apps/portage-2.3.99-r2 (/usr/bin/emerge -> ../lib/python-exec/python-exec2)
```

`/usr/bin/emerge` is a python script wrapped below python-exec2. python-exec2 forwards the execution of `/usr/bin/foo` to `/usr/lib/python-exec/pythonX.Y/foo` where pythonX.Y is one of the python implementations listed in preference order on `/etc/python-exec/python-exec.conf`. With this information, I've fould the my emerge script on `/usr/lib/python-exec/python3.6/emerge`.

```sh
equery b /usr/lib/python-exec/python3.6/emerge
# prints:
#   sys-apps/portage-2.3.99-r2 (/usr/lib/python-exec/python3.6/emerge)
```

### Listing files inside the installed package

The location `/var/db/pkg/<pachage>`, `/var/db/pkg/sys-apps/portage-2.3.99-r2/` in this case, contains information for every installed package. It containes a set of files about the installation.

Additionally, the 'f' or 'files' module of the equery command lists the installed files: `equery files|f [OPTIONS] <package>`. Try `--tree` to get an easy to read directory layout. You could also apply the `--filter` option to only find a certain type of file. For example, to find where executables were installed use `--filter=cmd`, or to quickly find the configuration file location try `--filter=conf`.

Example:

```sh
equery files --tree sys-apps/portage-2.3.99-r2 | less
```

### Extracting information from the package

You can run the following commands for extracting information about the installed package:
- `equery meta sys-apps/portage` - shows description, ChangeLog URL, maintainers, and project homepage - each package in the Portage tree provides at least some metadata. The 'm' or 'meta' module reads a file called 'metadata.xml' which must be included with all Portage tree packages. 'meta' does not read ebuilds, so it can only return version independent metadata. Since until now there had not been an easy way for users to view 'metadata.xml', and because package maintainers are only required to fill out a very small portion of the file (the amount of useful information depends on how much package maintainers decide to provide), there are still many packages without detailed metadata available. For more information about metadata.xml, see: https://devmanual.gentoo.org/ebuild-writing/misc-files/metadata/index.html. With no options, meta returns some basic useful information.
- `equery depgraph sys-apps/portage-2.3.99-r2` - shows the dependencies.
- `equery which sys-apps/portage-2.3.99-r2` - shows the path to the ebuild that would be used by Portage with the current configuration.
- `equery changes sys-apps/portage-2.3.99-r2` - shows the Gentoo ChangeLog entry for the latest installable version of the package.
- `emerge --info sys-apps/portage` - Produces a list of information to include in bug reports which aids the developers when fixing the reported problem. You should include this information when submitting a bug report. Expanded output can be obtained with the '--verbose' option.
- `bzless /usr/share/doc/portage-2.3.99-r2/README.bz2` - the README file can contain any kind of information, it is usually compressed with bz2 and that is why we use `bzless`.

### Finding the package

#### On the web page

TODO

You can either:

- Generate an URL using `https://packages.debian.org/{PAKAGE_NAME}` and your package name. For example, `https://packages.debian.org/lxqt-admin`.
- Generate an URL using `https://packages.debian.org/search?keywords={PAKAGE_NAME}` and your package name. For example, `https://packages.debian.org/search?keywords=lxqt-admin`.
- Going to `https://packages.debian.org/` and searching the package manually.

The page shows multiple options all related to an specific debian release (aka suite) codename like strech, buster, bullseye, bookworm and sid (the development distribution of Debian). You can get more information about Debian suites in [https://wiki.debian.org/DebianReleases](https://wiki.debian.org/DebianReleases) and also you can see what suite your distro is using by running `cat /etc/debian_version`.

The option "sid" ([Debian Unestable](https://wiki.debian.org/DebianUnstable)) is probably the best option. It is not strictly a release, but rather a "rolling development" version of the Debian distribution containing the latest packages that have been introduced into Debian.

For this example I can see the most up to date information about a package going to `https://packages.debian.org/sid/lxqt-admin`.

https://packages.gentoo.org/


            - [In Debian](#in-debian)
            - [In Ubuntu](#in-ubuntu)
        - [Finding the source spackage in Debian](#finding-the-source-spackage-in-debian)
            - [In Debian](#in-debian)
            - [In Ubuntu](#in-ubuntu)
        - [Finding the repository with files for packaging](#finding-the-repository-with-files-for-packaging)
            - [In Debian](#in-debian)
            - [In Ubuntu](#in-ubuntu)
        - [Finding the upstream original repository](#finding-the-upstream-original-repository)
            - [In Debian](#in-debian)
            - [In Ubuntu](#in-ubuntu)

### How to download a binary package

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

Also accorrding to the version 2 of the "Packages directory layout" (aka. PKGDIR layout), inside the top directory of the binary packages exists a manifest file called 'Packages'. This file acts as a cache for the metadata of all binary packages in the packages directory. The file is updated whenever Portage adds a binary package to the directory. Similarly, eclean updates it when it removes binary packages. If for some reason binary packages are simply deleted or copied into the packages directory, or the 'Packages' file gets corrupted or deleted, then it must be recreated using the 'emaint' command.

```sh
wget https://isshoni.org/pi64pie/Packages
```

New binary packages uses the XPAK format. XPAK format binary packages created by Portage have the file name ending with '.tbz2', these files consist of two parts:

* A '.tar.bz2' archive containing the files that will be installed on the system.
* A 'xpak' archive containing package metadata, the ebuild, and the environment file. See `man xpak` for a description of the format.

In 'app-portage/portage-utils' some tools exists that are able to split or create 'tbz2' and 'xpak' files.

Example:

First I checked the binary package host that will be used:

```sh
cat /etc/portage/make.conf | grep PORTAGE_BINHOST
# prints, for example:
#   PORTAGE_BINHOST="https://isshoni.org/pi64pie"
```

Check the versions of the package listed on the Packages file:

```sh
curl https://isshoni.org/pi64pie/Packages | grep 'CPV: kde-frameworks/kwindowsystem'
# prints:
#   CPV: kde-frameworks/kwindowsystem-5.63.0
#   CPV: kde-frameworks/kwindowsystem-5.64.0
#   CPV: kde-frameworks/kwindowsystem-5.65.0
#   CPV: kde-frameworks/kwindowsystem-5.66.0
#   CPV: kde-frameworks/kwindowsystem-5.67.0
#   CPV: kde-frameworks/kwindowsystem-5.68.0
#   CPV: kde-frameworks/kwindowsystem-5.69.0
#   CPV: kde-frameworks/kwindowsystem-5.70.0
#   CPV: kde-frameworks/kwindowsystem-5.71.0
#   CPV: kde-frameworks/kwindowsystem-5.72.0
#   CPV: kde-frameworks/kwindowsystem-5.73.0
#   CPV: kde-frameworks/kwindowsystem-5.74.0
```

Then run the fetch command:

```sh
# (1) tries to fetch version 5.70.0 because of the latest ebuild file that was found
emerge --debug --pretend --verbose --fetchonly kde-frameworks/kwindowsystem
# Calculating dependencies
#      Arg: kde-frameworks/kwindowsystem
#     Atom: kde-frameworks/kwindowsystem
#   ebuild: kde-frameworks/kwindowsystem-5.70.0::gentoo
#   binary: kde-frameworks/kwindowsystem-5.70.0::gentoo
# prints a lot, but URL of the binary package was:
#   https://isshoni.org/pi64pie/kde-frameworks/kwindowsystem-5.70.0.tbz2

# (2) similar to (1)
emerge --debug --pretend --verbose --fetchonly --getbinpkg kde-frameworks/kwindowsystem
# Calculating dependencies
#      Arg: kde-frameworks/kwindowsystem
#     Atom: kde-frameworks/kwindowsystem
#   ebuild: kde-frameworks/kwindowsystem-5.70.0::gentoo
#   binary: kde-frameworks/kwindowsystem-5.70.0::gentoo
# prints a lot, but URL of the binary package was:
#   https://isshoni.org/pi64pie/kde-frameworks/kwindowsystem-5.70.0.tbz2

# (3) tries to fetch version 5.74.0 because it is the latest bin package that was found
#   at /var/cache/edb/binhost/isshoni.org/pi64pie/Packages
emerge --debug --pretend --verbose --fetchonly --getbinpkgonly kde-frameworks/kwindowsystem
# Calculating dependencies
#      Arg: kde-frameworks/kwindowsystem
#     Atom: kde-frameworks/kwindowsystem
#   binary: kde-frameworks/kwindowsystem-5.74.0::gentoo
# prints a lot, but URL of the binary package was:
#   https://isshoni.org/pi64pie/kde-frameworks/kwindowsystem-5.74.0.tbz2
```

Download it. I will be downloading the version 5.70.0:

```sh
mkdir temp
curl https://isshoni.org/pi64pie/kde-frameworks/kwindowsystem-5.70.0.tbz2 -o temp/kwindowsystem-5.70.0.tbz2
cd temp
qtbz2 -s kwindowsystem-5.70.0.tbz2
ls
# prints:
#   kwindowsystem-5.70.0.tar.bz2  kwindowsystem-5.70.0.tbz2  kwindowsystem-5.70.0.xpak
```

Explore the content of the xpak file:

```sh
qxpak -l kwindowsystem-5.70.0.xpak
# prints:
#   BDEPEND
#   BUILD_TIME
#   CATEGORY
#   CBUILD
#   CFLAGS
#   CHOST
#   CXXFLAGS
#   DEFINED_PHASES
#   DEPEND
#   DESCRIPTION
#   EAPI
#   FEATURES
#   HOMEPAGE
#   INHERITED
#   IUSE
#   IUSE_EFFECTIVE
#   KEYWORDS
#   LDFLAGS
#   LICENSE
#   NEEDED
#   NEEDED.ELF.2
#   PF
#   PROVIDES
#   RDEPEND
#   REQUIRES
#   RESTRICT
#   SIZE
#   SLOT
#   USE
#   environment.bz2
#   kwindowsystem-5.70.0.ebuild
#   repository

# Extract a file called USE which contains the enabled USE flags for this package
qxpak -x kwindowsystem-5.70.0.xpak USE
less USE
```

Explore the content of the binary package:

```sh
mkdir kwindowsystem-5.70.0
tar xvf kwindowsystem-5.70.0.tar.bz2 -C kwindowsystem-5.70.0
tree kwindowsystem-5.70.0 | less
```

### How to download a source package

You can either:

(1) Look inside the ebuild and there you will see a line starting with 'SRC_URI'. This line has the files which emerge would download. You can download all the files from any mirror (https://www.gentoo.org/downloads/mirrors/), just look into the distfiles directory.

NOTE: Packages will automatically have their 'SRC_URI' components mirrored onto Gentoo mirrors, including those hosted on the official Gentoo Infrastructure (i.e. developer space at dev.gentoo.org). **When fetching, package manager checks Gentoo mirrors first before trying the original upstream location**, so maybe (2) would be the better procesure to get the real URL, probable from a mirror before the SRC_URI, where the package would be download.

Example:

```sh
equery which lxqt-base/lxqt-about
# prints:
#   /var/db/repos/gentoo/lxqt-base/lxqt-about/lxqt-about-0.15.0.ebuild
cat /var/db/repos/gentoo/lxqt-base/lxqt-about/lxqt-about-0.15.0.ebuild | egrep 'SRC_URI|EGIT_REPO_URI'
# prints:
#   EGIT_REPO_URI="https://github.com/lxqt/${PN}.git"
#   SRC_URI="https://github.com/lxqt/${PN}/releases/download/${PV}/${P}.tar.xz"
```

(2) Issue the command `emerge -pvf package` to get a list of the files which emerge would download, and then you can use this list and download it from one of the mirrors. Portage always tries the mirrors (the GENTOO_MIRRORS variable) from `/etc/portage/make.conf` first. `emerge -f <package>` will fetch the packages and put them in `/usr/portage/distfiles`.

The GENTOO_MIRRORS variable is set in `/usr/share/portage/config/make.globals`, but it can be overwritten with an entry in `/etc/portage/make.conf`.

```sh
cat /usr/share/portage/config/make.globals | grep GENTOO_MIRRORS
cat /etc/portage/make.conf | grep GENTOO_MIRRORS
emerge --pretend --verbose --fetchonly <package>
# or emerge -pvf <package>
```

You can use one of the following options to fetch the sources:

```
--fetchonly, -f
  Instead of doing any package building, just perform fetches for all packages (fetch things from SRC_URI based upon USE setting).

--fetch-all-uri, -F
  Instead of doing any package building, just perform fetches for all packages (fetch everything in SRC_URI regardless of USE setting).
```

Example:

First check the mirrors that emerge will be using to determine the final package URL.

```sh
# (1) the default list
cat /usr/share/portage/config/make.globals | grep GENTOO_MIRRORS
# prints:
#   GENTOO_MIRRORS="http://distfiles.gentoo.org"

# (2) the configured list
cat /etc/portage/make.conf | grep GENTOO_MIRRORS
# prints:
#   GENTOO_MIRRORS="http://gentoo.osuosl.org/ http://trumpetti.atm.tut.fi/gentoo/ http://distfiles.gentoo.org"

# (3) the SRC_URI
equery which lxqt-base/lxqt-about
# prints:
#   /var/db/repos/gentoo/lxqt-base/lxqt-about/lxqt-about-0.15.0.ebuild
cat /var/db/repos/gentoo/lxqt-base/lxqt-about/lxqt-about-0.15.0.ebuild | egrep 'SRC_URI|EGIT_REPO_URI'
# prints:
#   EGIT_REPO_URI="https://github.com/lxqt/${PN}.git"
#   SRC_URI="https://github.com/lxqt/${PN}/releases/download/${PV}/${P}.tar.xz"
```

Then run the fetch command:

```sh
emerge --pretend --verbose --fetchonly lxqt-base/lxqt-about
# prints a lot, but we can see the list of URLs that emerge will check:
#   http://gentoo.osuosl.org/distfiles/ef/lxqt-about-0.15.0.tar.xz
#   http://trumpetti.atm.tut.fi/gentoo/distfiles/lxqt-about-0.15.0.tar.xz
#   http://distfiles.gentoo.org/distfiles/ef/lxqt-about-0.15.0.tar.xz
#   https://github.com/lxqt/lxqt-about/releases/download/0.15.0/lxqt-about-0.15.0.tar.xz
```

From the above command we can see the emerge command will try to fetch the source package from the following URLs in priority order:

* first from the URL determined using 1st mirror: http://gentoo.osuosl.org/distfiles/ef/lxqt-about-0.15.0.tar.xz
* from the URL determined using 2nd mirror: http://trumpetti.atm.tut.fi/gentoo/distfiles/lxqt-about-0.15.0.tar.xz
* from the URL determined using 3rd mirror: http://distfiles.gentoo.org/distfiles/ef/lxqt-about-0.15.0.tar.xz
* finally from the URL determined using SRC_URI: https://github.com/lxqt/lxqt-about/releases/download/0.15.0/lxqt-about-0.15.0.tar.xz

#### The exact location of the source package in a mirror

From the above example we can see that some mirrors could have the source package file in a different location other than the common top 'distfiles' directory on `<mirror-url>/distfiles/<package-src-file>`.

For mirror 1, the URL for the lxqt-about package was `http://gentoo.osuosl.org/distfiles/ef/lxqt-about-0.15.0.tar.xz`.

The "GLEP 75: Split distfile mirror directory structure" specification says that a mirror should include a 'layout.conf' file in the top distfile directory: `<mirror-url>/distfiles/layout.conf`.

For mirror 1, the URL of the 'layout.conf' is `http://gentoo.osuosl.org/distfiles/layout.conf`. And the content is:

```ini
[structure]
0=filename-hash BLAKE2B 8
1=flat
```

The '[structure]' section defines one or more repository structure definitions using non-negative sequential integer keys. The definition with the '0' key is the most preferred structure. The package manager should ignore any formats it does not recognize. If this section is not present, the package manager should behave as if only flat structure were specified.

The following structure definitions are supported:

* 'flat' to indicate the traditional flat structure where all distfiles are located in the top directory,
* 'filename-hash <algorithm> <cutoffs>' to indicate the "filename hash structure". '<algorithm>' must correspond to a valid Manifest hash name. An informational list of hashes is included in GLEP 74, and the policies for introducing new hashes are covered by GLEP 59. The '<cutoffs>' list specifies one or more integers separated by colons (:), indicating the number of bits (starting with the most significant bit) of the hash used to form subsequent subdirectory names. For example, the list of 4:8 would indicate that top-level directory names are formed using 4 most significant bits of the hash (resulting in 2⁴ = 16 directories), and each of these directories would have subdirectories formed using the next 8 bits of the hash (resulting in 2⁸ = 256 subdirectories each).

The exact algorithm for determining the distfile location for 'filename-hash' is as follows:

1. Let the distfile filename be F.
2. Compute the hash of F and store its binary value as H.
3. For each integer C in cutoff list:
  a. Remove the C most significant bits from H and store them as V.
  b. Convert V into hexadecimal notation (digits 0 to 9 and lowercase letters a to f), left padded with zeros to C/4 digits (rounded up), and append it to the path, followed by the path separator.
4. Finally, append F to the obtained path.

In particular, note that when using nested directories the subdirectories do not repeat the hash bits used in parent directory.

Then, continuing with our example we can conclude the 'ef' subdirectory was determined by taking the first 8 bits of the BLAKE2B algorithm calculated for the filename 'lxqt-about-0.15.0.tar.xz':

```sh
echo -n "lxqt-about-0.15.0.tar.xz" | b2sum
# prints:
#   efc107e6433c02ce8009fa1c52dc489178d04e26691a0c799e8abf13e62f4ced708e86a40db98e8e17080c5863b2228c1cc472d525522c3bc5621f1fa8faa6be
# look that 'ef' is the first 8 bits.
```
