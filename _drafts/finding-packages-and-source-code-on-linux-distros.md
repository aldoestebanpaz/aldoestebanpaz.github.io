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

## dpkg and apt based (Debian and Ubuntu)

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

### Finding the binary package in Debian

Note "package binary" and "package" are synonyms.

You could:

- Generate an URL using `https://packages.debian.org/name` and your package name. For example, `https://packages.debian.org/lxqt-admin`.
- Generate an URL using `https://packages.debian.org/search?keywords=name` and your package name. For example, `https://packages.debian.org/search?keywords=lxqt-admin`.
- Going to `https://packages.debian.org/` and searching the package manually.

The page shows multiple options all related to an specific debian release (aka suite) codename like strech, buster, bullseye, bookworm and sid. You can get more information about Debian suites in [https://wiki.debian.org/DebianReleases](https://wiki.debian.org/DebianReleases) and also you can see what suite your distro is using by running `cat /etc/debian_version`.

The option "sid" ([Debian Unestable](https://wiki.debian.org/DebianUnstable)) is probably the best option. It is not strictly a release, but rather a rolling development version of the Debian distribution containing the latest packages that have been introduced into Debian.

For this example I can see the most up to date information about a package going to `https://packages.debian.org/sid/lxqt-admin`.

### Finding the package source in Debian

A "package binary" (or just "package") is different than a "package source". A package source could be the source of more than one package, in other words one or more binary packages can be derived from a single package source. For example, the page for the source package "shadow" (the shadow tools aka. shadow-utils, https://packages.debian.org/source/sid/shadow) shows that binary packages "libsubid-dev", "libsubid4", "login", "passwd" and "uidmap" are built from it.

In our example there is a 1:1 relationship between the package name and the source package name, the package name and the source package name are the same. But from package source lxqt-admin derives lxqt-admin and lxqt-admin-l10n.

You can get the link to the source package from a specific package page in packages.debian.org. For example, if you go to `https://packages.debian.org/sid/lxqt-admin` you will see a link to the source package page in the beginning: `https://packages.debian.org/source/sid/lxqt-admin`.

Other alternative is opening the "Developer Information" link that appears in the same page in the right side. In this example it is `https://tracker.debian.org/pkg/lxqt-admin`. This page shows the link to the source package too, `https://packages.debian.org/src:lxqt-admin` in this case.

`https://packages.debian.org/src:lxqt-admin` shows the options for the different debian releases, while `https://packages.debian.org/source/sid/lxqt-admin` is the source package specific for the sid (unestable) suite.

### Finding the repository with packaging files

You can get the link:
- Opening the source package page `https://packages.debian.org/source/sid/lxqt-admin` and clicking the "Debian Source Repository" link.
- Opening the Developer Information page `https://tracker.debian.org/pkg/lxqt-admin` and clicking the VCS's Browse link.

The link in this example is `https://salsa.debian.org/lxqt-team/lxqt-admin`.

### Finding the upstream (original) repository

You can get the link:
- Opening the binary package page `https://packages.debian.org/sid/lxqt-admin` and clicking the "Homepage" link.
- Opening the source package page `https://packages.debian.org/source/sid/lxqt-admin` and clicking the "Homepage" link.
- Opening the Developer Information page `https://tracker.debian.org/pkg/lxqt-admin` and clicking the "homepage" link.
- Following one of the options listed in the "Extracting information from the package" section above.

The link in this example is `https://github.com/lxqt/lxqt-admin`.
