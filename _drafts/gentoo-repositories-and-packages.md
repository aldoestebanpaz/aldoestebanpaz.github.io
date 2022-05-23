# Gentoo repositories and packages

## Repositories, aka. ebuild repository

The Gentoo ebuild repository contains ebuild files that tell Portage how to build and install each package. The ebuilds come with metadata, dependency information, and everything else needed to get a package in working order.

### How to list the configured repositories

'portageq' can show all the repositories currently configured with portage:

```sh
portageq repos_config /
```

### The default repository

The 'Gentoo ebuild repository' is the Gentoo Linux's primary and official ebuild repository, and is where all the packages/ebuilds come from by default. It is maintained on the [gitweb.gentoo.org](https://gitweb.gentoo.org/repo/gentoo.git/tree) server, and gets synchronized to local machines in `/var/db/repos/gentoo` to be available to Portage.

Additional ebuild repositories, such as GURU (an ebuild repository entirely maintained by Gentoo users), can be configured with Portage to provide even more packages.

### Overlays

Portage will install the latest available version of a package from any configured ebuild repository, by default. If the latest available version is provided by several ebuild repositories, it will be chosen according to a set order of priority - hence the colloquial name 'overlay'.

## Packages

On Gentoo, software packages are managed using ebuild files.

An ebuild file is a text file that is usually stored in a repository and which identifies a specific software package and tells the Gentoo package manager how to handle it. The file use a bash-like syntax style and is standardized through the Package Manager Specification by adhering to a specific EAPI version.
