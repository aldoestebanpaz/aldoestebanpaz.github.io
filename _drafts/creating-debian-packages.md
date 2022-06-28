- [Creating a debian package](#creating-a-debian-package)
  - [(1) Download the upstream tarball and test it](#1-download-the-upstream-tarball-and-test-it)
  - [(2) Debianize the source tree](#2-debianize-the-source-tree)
    - [Requirement: the package-version/ directory name format](#requirement-the-package-version-directory-name-format)
    - [Requirement: the ../package-version.tar.gz tarball is required](#requirement-the-package-versiontargz-tarball-is-required)
    - [Setting the name and email of the package maintainer before running debmake](#setting-the-name-and-email-of-the-package-maintainer-before-running-debmake)
    - [Other template files](#other-template-files)
    - [Automate the copy/download unpack and debianization with debmake](#automate-the-copydownload-unpack-and-debianization-with-debmake)
  - [(3) Customize the template files in debian/](#3-customize-the-template-files-in-debian)
  - [(4) Build the package](#4-build-the-package)
    - [(4.1) Trying and failing](#41-trying-and-failing)
    - [(4.2) Fixing the dh's 'clean' sequence, trying and failing again](#42-fixing-the-dhs-clean-sequence-trying-and-failing-again)
      - [What happened?](#what-happened)
    - [(4.3) Overriding 'dh_autoreconf', triying and failing again](#43-overriding-dh_autoreconf-triying-and-failing-again)
      - [Generated files in the parent directory (..)](#generated-files-in-the-parent-directory-)
        - [View the content of a generated Debian package](#view-the-content-of-a-generated-debian-package)
        - [The content of the .debian.tar.xz](#the-content-of-the-debiantarxz)
      - [Generated files in the build directory (.)](#generated-files-in-the-build-directory-)
      - [Example content in 'debian/hello.substvars'](#example-content-in-debianhellosubstvars)
    - [(4.4) Fixing errors found by 'lintian'](#44-fixing-errors-found-by-lintian)
      - [Issues at 'debian/README.Debian'](#issues-at-debianreadmedebian)
      - [Issues at 'debian/control'](#issues-at-debiancontrol)
      - [Issues at 'debian/changelog'](#issues-at-debianchangelog)
        - [The distribution value in a changelog entry is important](#the-distribution-value-in-a-changelog-entry-is-important)
        - [Ignoring lintian's 'initial-upload-closes-no-bugs' tag](#ignoring-lintians-initial-upload-closes-no-bugs-tag)
        - [Modifying changelog entries with 'debchange' (dch)](#modifying-changelog-entries-with-debchange-dch)
      - [Issues at 'debian/copyright'](#issues-at-debiancopyright)
        - [License checkers](#license-checkers)
  - [(5) Package signing](#5-package-signing)
    - ['debuild' and package signing](#debuild-and-package-signing)
    - [Creating a GPG key](#creating-a-gpg-key)
      - [Create a backup of your GPG keys](#create-a-backup-of-your-gpg-keys)
    - [GPG signing Debian source packages (.dsc) and changes (.changes) files](#gpg-signing-debian-source-packages-dsc-and-changes-changes-files)
    - [GPG signing Debian binary packages (.deb) files](#gpg-signing-debian-binary-packages-deb-files)
      - [Verifying GPG signatures of .deb package files](#verifying-gpg-signatures-of-deb-package-files)
      - [dpkg support for 'debsig-verify'](#dpkg-support-for-debsig-verify)
      - ['dpkg-sig', another .deb file signing tool](#dpkg-sig-another-deb-file-signing-tool)
      - [dpkg support for dpkg-sig](#dpkg-support-for-dpkg-sig)
    - [GPG signatures for APT repository metadata](#gpg-signatures-for-apt-repository-metadata)
  - [(6) Functional tests with DEP-8](#6-functional-tests-with-dep-8)
    - [Creating a simple automated as-installed test](#creating-a-simple-automated-as-installed-test)
    - [Running the tests](#running-the-tests)
  - [Files in 'debian/' directory](#files-in-debian-directory)
    - [The minimum required for packaging](#the-minimum-required-for-packaging)
    - ['debian/compat'](#debiancompat)
    - ['debian/README.source' and 'debian/README.Debian'](#debianreadmesource-and-debianreadmedebian)
    - ['debian/*.docs'](#debiandocs)
    - ['debian/watch'](#debianwatch)
  - [What are the debhelper commands that 'dh' will execute for a given sequence?](#what-are-the-debhelper-commands-that-dh-will-execute-for-a-given-sequence)
  - [Skipping signing](#skipping-signing)
  - [Each source package may generate several binary packages](#each-source-package-may-generate-several-binary-packages)
  - [Each modification in a Debian package is the consequence of a modification made to the source package](#each-modification-in-a-debian-package-is-the-consequence-of-a-modification-made-to-the-source-package)
  - [Source only maintainer uploads since Debian 10 Buster](#source-only-maintainer-uploads-since-debian-10-buster)
  - [The build logs in buildd](#the-build-logs-in-buildd)
  - [The 'wrap-and-sort' command](#the-wrap-and-sort-command)
  - [The 'quilt' command](#the-quilt-command)
  - [Alternatives for binary package building](#alternatives-for-binary-package-building)
    - [Low level tools](#low-level-tools)
    - [High level tools](#high-level-tools)
  - [Debian packages - native vs non-native](#debian-packages---native-vs-non-native)
  - [The 'git-buildpackage' package](#the-git-buildpackage-package)
  - [Reference](#reference)

# Creating a debian package

At this guide I will be packaging a simple application called 'GNU Hello' which has been posted on [GNU.org](https://www.gnu.org/software/hello/), development project at [savannah.gnu.org](https://savannah.gnu.org/projects/hello/).


## (1) Download the upstream tarball and test it

We have to get the released tar from upstream (we call the authors of applications "upstream") and check that it compiles and runs.

```sh
# Download
wget -O hello-2.12.tar.gz "https://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz"


# Unpack
tar zxf hello-2.12.tar.gz


# Configure
cd hello-2.12
./configure
# ...
# config.status: creating Makefile
# config.status: creating src/Makefile
# ...
echo $?
# 0


# Run Make (shorthand for `make all`)
make
# ...


# Install (shorthand for `make install-exec install-data`)
sudo make install


# Test the package
hello


# Erase anything both `make` and `./configure` created, including the Makefile's
make distclean
```

## (2) Debianize the source tree

**Many packages are packaged using only a text editor while imitating how other similar packages are packaged and consulting how the 'Debian policy' requires us to do**. This seems to me the most popular method for the real-life packaging activity.

The 'debmake' command is a helper script that debianizes the upstream source tree by adding template files only in the debian directory. It is intended to replace functions offered historically by the 'dh_make' command.

Fetures:

- It always sets most of the obvious option states and values to reasonable defaults.
- It generates the upstream tarball and its required symlink if they are missing.
- It doesn't overwrite the existing configuration files in the **debian/** directory.
- It supports the **multiarch** package.
- It creates good template files such as the **debian/copyright** file compliant with **DEP-5**.

NOTE: If the debmake command is invoked with the -T option, more verbose comments are generated for the template files.

With the folowing command, 'debmake' will:
- Creates the hello_2.12.orig.tar.gz (note the underscore character) symlink pointing to the downloaded package,
- Temporarily change the workspace directory to hello-2.12/,
- Debianize it creating the debian/ directory,
- Runs the 'wrap-and-sort' command.

```sh
cd hello-2.12

debmake

tree ..
# .
# ├── hello-2.12
# ...
#     └── debian
#         ├── changelog
#         ├── control
#         ├── copyright
#         ├── patches
#         │   └── series
#         ├── README.Debian
#         ├── rules
#         ├── source
#         │   ├── format
#         │   └── local-options
#         └── watch
# ├── hello_2.12.orig.tar.gz -> hello-2.12.tar.gz
# └── hello-2.12.tar.gz
```

Since no options are specified, the 'debmake' command chooses reasonable default values for you:
- The source package name: hello
- The upstream version: 2.12
- The binary package name: hello
- The Debian revision: 1
- The package type: bin (the ELF binary executable package)
- The -x option: -x1 (default for the single binary package)

For other alternative packaging helper tools, see:
- [Debian wiki: AutomaticPackagingTools](https://wiki.debian.org/AutomaticPackagingTools) — Extensive comparison of packaging helper scripts
- [Debian wiki: CopyrightReviewTools](https://wiki.debian.org/CopyrightReviewTools) — Extensive comparison of copyright review helper scripts

### Requirement: the package-version/ directory name format

'debmake' without parameters needs to be invoked inside a directory with the **package-version**/ name format. If the directory doesn't have this format, you will see a message like it:

```
...
E: invalid parent directory for setting package/version: hello_2.12
E: rename parent directory to "package-version".
```

### Requirement: the ../package-version.tar.gz tarball is required

'debmake' without parameters requires of the upstream tarball in the parent directory. The upstream's tarball must have the same name as the working directory (../package-version.tar.gz). If the tarball doesn't have this format, you will see a message like it:

```
E: missing "hello-2.12.tar.gz".
```

### Setting the name and email of the package maintainer before running debmake

We could set the maintainer name and email by running `debmake -e aldo.paz@noemail.org -f 'Aldo Paz'` where:

```
-f "firstname lastname", --fullname "firstname lastname"
        set the fullname.

        The default is taken from the value of the environment variable $DEBFULLNAME.
-e foo@example.org, --email foo@example.org
        set e-mail address.

        The default is taken from the value of the environment variable $DEBEMAIL.
```

### Other template files

```
-x n, --extra n
           generate extra configuration files as templates.

           The number n changes which configuration templates are generated.

           ·   -x0: bare minimum configuration files. (default if these files exist already)

           ·   -x1: ,, + desirable configuration files. (default for new packages)

           ·   -x2: ,, + interesting configuration files. (recommended for experts, multi binary
               aware)

           ·   -x3: ,, + unusual configuration template files with the extra .ex suffix to ease
               their removal. (recommended for new users) To use these as configuration files,
               rename their file names into ones without the .ex suffix.

           ·   -x4: ,, + copyright file examples.
```

### Automate the copy/download unpack and debianization with debmake

With the folowing command, 'debmake' will:
- Copy or download the upstream package in the current directory,
- Unpack it,
- Rename the created directory to hello-2.12/ (as specified by --package and --upstreamversion, note the dash character) if necessary,
- Creates the hello_2.12.orig.tar.gz (note the underscore character) symlink pointing to the downloaded package,
- Temporarily change the workspace directory to hello-2.12/,
- Debianize it creating the debian/ directory,
- Runs the 'wrap-and-sort' command.

```sh
mkdir tmp
cd tmp

debmake --archive https://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz \
  --package hello \
  --upstreamversion 2.12

tree
# .
# ├── hello-2.12
# ...
#     └── debian
#         ├── changelog
#         ├── control
#         ├── copyright
#         ├── patches
#         │   └── series
#         ├── README.Debian
#         ├── rules
#         ├── source
#         │   ├── format
#         │   └── local-options
#         └── watch
# ├── hello_2.12.orig.tar.gz -> hello-2.12.tar.gz
# └── hello-2.12.tar.gz
```

## (3) Customize the template files in debian/

At this point the maintainer should customize the template files generated inside 'debian/' by 'debmake'. We will skip the customization here to show the possible errors that 'lintian', the static analysis tool for Debian packages, could throw when building a package.

## (4) Build the package

NOTE: Much of the package building work is done by a series of scripts called 'debhelper'. The exact behaviour of debhelper changes with new major versions, the 'compat' number in the 'debian/control' file instructs 'debhelper' which version to act as. You will generally want to set this to the most recent version at the moment of packaging.

Here I will be using 'debuild', a wrapper script of the 'dpkg-buildpackage' command to build the Debian binary package ('dpkg-buildpackage' is the official command for it) under the proper environment variables. For normal binary build, it executes roughly:

- `dpkg-source --before-build` - apply Debian patches, unless they are already applied.
- `fakeroot debian/rules clean` - clean the build directory (aka. '<\<PKGBUILDDIR\>>') before build.
- `dpkg-source -b .` - **build the Debian source package** and *.dsc file.
- `fakeroot debian/rules build` - build the upstream package.
- `fakeroot debian/rules binary` - **build Debian binary packages**.
- `dpkg-genbuildinfo` -  generate a *.buildinfo file.
- `dpkg-genchanges` -  generate a *.changes file.
- `fakeroot debian/rules clean` - clean the build directory.
- `dpkg-source --after-build` - unapply Debian patches, if they are applied during --before-build.
- `debsign` - sign the *.dsc and *.changes files.

See `man debuild` and `man dpkg-buildpackage` for more details.

### (4.1) Trying and failing

Inside the source tree, now known as the build directory (aka. '<\<PKGBUILDDIR\>>'), we can execute 'debuild'.

```sh
cd hello-2.12

# Build
debuild
#  dpkg-buildpackage -us -uc -ui
# dpkg-buildpackage: info: source package hello
# dpkg-buildpackage: info: source version 2.12-1
# dpkg-buildpackage: info: source distribution UNRELEASED
# dpkg-buildpackage: info: source changed by Aldo <>
#  dpkg-source --before-build .
# dpkg-buildpackage: info: host architecture amd64
#  fakeroot debian/rules clean
# dh clean --with autoreconf
#    dh_auto_clean
#         make -j4 distclean
# make[1]: Entering directory '/home/apaz/packaging-example/hello-2.12'
# There seems to be no Makefile in this directory.
# You must run ./configure before running 'make'.
# make[1]: *** [GNUmakefile:108: abort-due-to-no-makefile] Error 1
# make[1]: Leaving directory '/home/apaz/packaging-example/hello-2.12'
# dh_auto_clean: error: make -j4 distclean returned exit code 2
# make: *** [debian/rules:9: clean] Error 25
# dpkg-buildpackage: error: fakeroot debian/rules clean subprocess returned exit status 2
# debuild: fatal error at line 1182:
# dpkg-buildpackage -us -uc -ui failed

# Logs
less ../hello_2.12-1_amd64.build
```

The 'clean' target was invoked in 'debian/rules', executing the 'dh clean --with autoreconf' command. The dh's 'clean' sequence eventually calls the 'dh_auto_clean' command, which finally called the 'make -j4 distclean' command.

'autoreconf' in this case is one of many addons availale for debhelper. You list all the available addons with `dh --list`.

The 'make' command obviously fails because no Makefile exists without running './configure' first.

See `man 7 debhelper`, `man dh`, `man dh_auto_clean` and `man 7 dh-autoreconf` for more details. Also you could run `dh <sequence> --no-act` to see what commands are included in a sequence, without actually doing anything.

### (4.2) Fixing the dh's 'clean' sequence, trying and failing again

We have to override the default execution of the 'dh_auto_clean' in the 'debian/rules':

```makefile
#!/usr/bin/make -f
%:
	dh $@

override_dh_auto_clean:
	[ ! -f Makefile ] || dh_auto_clean

```

This fix says "if the 'Makefile' file doesn't exists, then skip the 'dh_auto_clean' command".

If we try building again:

```sh
debuild

# Logs
less ../hello_2.12-1_amd64.build
#  dpkg-buildpackage -us -uc -ui
# dpkg-buildpackage: info: source package hello
# dpkg-buildpackage: info: source version 2.12-1
# dpkg-buildpackage: info: source distribution UNRELEASED
# dpkg-buildpackage: info: source changed by Aldo <>
#  dpkg-source --before-build .
# dpkg-buildpackage: info: host architecture amd64
#  fakeroot debian/rules clean
# dh clean --with autoreconf
#    debian/rules override_dh_auto_clean
# make[1]: Entering directory '/home/apaz/packaging-example/hello-2.12'
# [ ! -f Makefile ] || dh_auto_clean
# make[1]: Leaving directory '/home/apaz/packaging-example/hello-2.12'
#    dh_clean
#  dpkg-source -b .
# dpkg-source: info: using source format '3.0 (quilt)'
# dpkg-source: info: building hello using existing ./hello_2.12.orig.tar.gz
# dpkg-source: info: building hello in hello_2.12-1.debian.tar.xz
# dpkg-source: info: building hello in hello_2.12-1.dsc
#  debian/rules build
# dh build --with autoreconf
#    dh_update_autotools_config
#    dh_autoreconf
# ...
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# ...
#    dh_auto_configure
# 	./configure --build=x86_64-linux-gnu --prefix=/usr --includedir=\${prefix}/include --mandir=\${prefix}/share/man --infodir=\${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-option-checking --disable-silent-rules --libdir=\${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking
# ...
#    dh_auto_build
# 	make -j4
# ...
# *** error: gettext infrastructure mismatch: using a Makefile.in.in from gettext version 0.18 but the autoconf macros are from gettext version 0.20
# ...
# dh_auto_build: error: make -j4 returned exit code 2
# ...
# dpkg-buildpackage: error: debian/rules build subprocess returned exit status 2
```

It failed again.

#### What happened?

An 'autoreconf' was attempted and it failed because of a bug in the distribution package that didn't found the 'build-aux/git-version-gen' file. 'configure.ac' references 'build-aux/git-version-gen' and appears to use it unconditionally.

So when `dh_autoreconf` does its job of regenerating the build scripts from configure.ac, then it will trigger those warnings because of the above.

`dh_autoreconf`, the program that executes 'autoreconf' behind scenes, was executed by `dh build` as part of the 'build' sequence. This sequence can be listed in the following way:

```sh
dh build --no-act
#    dh_testdir
#    dh_update_autotools_config
#    dh_autoreconf
#    dh_auto_configure
#    dh_auto_build
#    dh_auto_test
#    create-stamp debian/debhelper-build-stamp
```

Seems like the `dh_autoreconf` command is part of the 'build' sequence since compat 10, and we are using compat 12 (it is specified as 'debhelper-compat (= 12)' in the Build-Depends section in 'debian/control').

Possible options to fix this issue are:
- Have upstream include build-aux/git-version-gen in the source though it may need git and a git repo to work properly, so this might only make sense from the PoV of ensuring that we have the complete source of the build scripts.
- Patch configure.ac to not rely on the build-aux/git-version-gen script.
- Skip reconfiguration by overriding dh_autoreconf via an empty override_dh_autoreconf target.
- Skip dh_autoreconf by changing to compat 9.

At the next section we will solve this use through the the third option, by overriding 'dh_autoreconf'.

Reference: [Bug#928889: debhelper: dh_autoreconf says "sh: 1: build-aux/git-version-gen: not found"](https://www.mail-archive.com/debian-bugs-dist@lists.debian.org/msg1679890.html).

### (4.3) Overriding 'dh_autoreconf', triying and failing again

Here is the new version of 'debian/rules', this time skipping ''dh_autoreconf' by overriding it with an empty target:

```sh
%:
	dh $@ --with autoreconf

override_dh_auto_clean:
	[ ! -f Makefile ] || dh_auto_clean

# skip processing configure.ac because missing build-aux/git-version-gen.
override_dh_autoreconf:
```

Now trying to build again:

```sh
debuild

# Logs
less ../hello_2.12-1_amd64.build
#  dpkg-buildpackage -us -uc -ui
# dpkg-buildpackage: info: source package hello
# dpkg-buildpackage: info: source version 2.12-1
# dpkg-buildpackage: info: source distribution UNRELEASED
# dpkg-buildpackage: info: source changed by Aldo <>
#  dpkg-source --before-build .
# dpkg-buildpackage: info: host architecture amd64
#  fakeroot debian/rules clean
# dh clean --with autoreconf
#    debian/rules override_dh_auto_clean
# make[1]: Entering directory '/home/apaz/packaging-example/hello-2.12'
# [ ! -f Makefile ] || dh_auto_clean
# make[1]: Leaving directory '/home/apaz/packaging-example/hello-2.12'
#    dh_clean
#  dpkg-source -b .
# dpkg-source: info: using source format '3.0 (quilt)'
# dpkg-source: info: building hello using existing ./hello_2.12.orig.tar.gz
# dpkg-source: info: building hello in hello_2.12-1.debian.tar.xz
# dpkg-source: info: building hello in hello_2.12-1.dsc
#  debian/rules build
# dh build --with autoreconf
#    dh_update_autotools_config
#    dh_auto_configure
# 	./configure --build=x86_64-linux-gnu --prefix=/usr --includedir=\${prefix}/include --mandir=\${prefix}/share/man --infodir=\${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-option-checking --disable-silent-rules --libdir=\${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking
# ...
#    dh_auto_build
# 	make -j4
# ...
# (CDPATH="${ZSH_VERSION+.}:" && cd . && /bin/bash /home/apaz/packaging-example/hello-2.12/build-aux/missing autoheader)
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# sh: 1: build-aux/git-version-gen: not found
# rm -f stamp-h1
# ...
# make  all-recursive
# ...
# make  check-recursive
# ...
# make  check-TESTS
# ...
# PASS: tests/hello-1
# PASS: tests/greeting-1
# PASS: tests/atexit-1
# SKIP: tests/greeting-2
# PASS: tests/traditional-1
# PASS: tests/operand-1
# PASS: tests/last-1
# ============================================================================
# Testsuite summary for GNU Hello 2.12.1-6fe9
# ============================================================================
# # TOTAL: 7
# # PASS:  6
# # SKIP:  1
# # XFAIL: 0
# # FAIL:  0
# # XPASS: 0
# # ERROR: 0
# ============================================================================
# ...
#    create-stamp debian/debhelper-build-stamp
#  fakeroot debian/rules binary
# dh binary --with autoreconf
#    dh_testroot
#    dh_prep
#    dh_auto_install
# 	make -j4 install DESTDIR=/home/apaz/packaging-example/hello-2.12/debian/hello AM_UPDATE_INFO_DIR=no
# ...
# make  install-recursive
# ...
#    dh_installdocs
#    dh_installchangelogs
#    dh_installman
#    dh_perl
#    dh_link
#    dh_strip_nondeterminism
# ...
#    dh_compress
#    dh_fixperms
#    dh_missing
#    dh_dwz
#    dh_strip
# 9f8a74c319630131487b35f5959d3711ed35854a
#    dh_makeshlibs
#    dh_shlibdeps
#    dh_installdeb
#    dh_gencontrol
#    dh_md5sums
#    dh_builddeb
# dpkg-deb: construyendo el paquete `hello' en `../hello_2.12-1_amd64.deb'.
# dpkg-deb: construyendo el paquete `hello-dbgsym' en `debian/.debhelper/scratch-space/build-hello/hello-dbgsym_2.12-1_amd64.deb'.
# 	Renaming hello-dbgsym_2.12-1_amd64.deb to hello-dbgsym_2.12-1_amd64.ddeb
#  dpkg-genbuildinfo
#  dpkg-genchanges  >../hello_2.12-1_amd64.changes
# dpkg-genchanges: info: including full source code in upload
#  dpkg-source --after-build .
# dpkg-buildpackage: info: full upload (original source is included)
# Now running lintian hello_2.12-1_amd64.changes ...
# E: hello: bogus-mail-host-in-debian-changelog Aldo <>
# E: hello: changelog-is-dh_make-template
# E: hello: copyright-file-contains-full-gpl-license
# E: hello source: malformed-contact Maintainer Aldo <>
# E: hello: malformed-contact Maintainer Aldo <>
# E: hello-dbgsym: malformed-contact Maintainer Aldo <>
# E: hello changes: malformed-contact Changed-By Aldo <>
# E: hello changes: malformed-contact Maintainer Aldo <>
# E: hello: section-is-dh_make-template
# W: hello source: bad-homepage <insert the upstream URL, if relevant>
# W: hello: bad-homepage <insert the upstream URL, if relevant>
# W: hello: copyright-has-url-from-dh_make-boilerplate
# W: hello: initial-upload-closes-no-bugs
# W: hello: readme-debian-contains-debmake-template
# W: hello source: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello source: syntax-error-in-dep5-copyright debian/copyright: Continuation line not in paragraph (line 2178). Missing a dot on the previous line?
# W: hello source: useless-autoreconf-build-depends dh-autoreconf
# W: hello: wrong-bug-number-in-closes #nnnn in the installed changelog (line 3)
# Finished running lintian.
```

NOTE that, even when in some part of the build process the file 'build-aux/git-version-gen' stills is checked and not found, the compilation and testing pass and finally the binary package is created.

Even if it builds the .deb binary package, the packaging may have bugs. Many errors can be automatically detected by 'lintian' (like the logs above where 'lintian' was executed using the .changes file). In the next step of the process we will fix the errors throwed by 'lintian', the package checker for Debian packages

#### Generated files in the parent directory (..)

Now the generated files in the parent directory are the following.

Files for analysis and troubleshooting:

- 'hello_2.12-1_amd64.build' - A logs file. We were using it anove to analyze the messages of the building process.
- 'hello_2.12-1_amd64.buildinfo' - A Debian control file describing the build environment and the build artifacts. It has several goals which are related to each other. (1) Can be useful for forensics/debugging because it records information about the system environment used during a particular build -- packages installed (toolchain, etc), system architecture, etc. (2) It can also be used to try to recreate (partially or in full) the system environment when trying to reproduce a particular build. See `man 5 deb-buildinfo` and the [ReproducibleBuilds BuildinfoFiles ](https://wiki.debian.org/ReproducibleBuilds/BuildinfoFiles) wiki for more details.

For uploading:

- 'hello_2.12-1_amd64.changes' - The Debian upload control file used both for source-only or source+binary. The .changes files are used by the Debian archive maintenance software to process updates to packages, because each Debian upload is composed of a .changes control file. They consist of a single paragraph, possibly surrounded by a PGP signature. That paragraph contains information from the debian/control file and other data about the source package gathered via debian/changelog and debian/rules. See `man 5 deb-changes`, `man dpkg-genchanges` and [5.5. Debian changes files – .changes in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-controlfields.html#debian-changes-files-changes) for details.

Source-package related:

- 'hello_2.12-1.dsc' - The Debian Source Control file. Describes the source package and indicates which other files are part thereof. It is signed by its maintainer, which guarantees authenticity. This file is generated by 'dpkg-source' when it builds the source archive. See `man dpkg-source` and `man 5 dsc` for details.
- 'hello_2.12-1.debian.tar.xz' - The debianization patch - Historically it was known as the '.diff.gz' file in 1.0 format, it contains the Debian maintainer generated/modified contents; the 'debian/' directory and patches. A debian.tar.xz archive contains the compiling instructions and a set of upstream patches contributed by the package maintainer. These last are recorded in a format compatible with quilt — a tool that facilitates the management of a series of patches.

Binary-package related:

- 'hello_2.12-1_amd64.deb' - The Debian binary package. See `man dh_builddeb`, `man dpkg-deb` and `man 5 deb` for details.
- 'hello-dbgsym_2.12-1_amd64.ddeb' - The dbgsym package (aka. debug package). See `man dh_builddeb` and `man dh_strip` for details.

##### View the content of a generated Debian package

We can list the content of a .deb package with 'lesspipe', or of all .deb packages listed in the .changes file with 'debc':

```sh
lesspipe ../hello_2.12-0ubuntu1_amd64.deb
#...

debc ../hello_2.12-0ubuntu1_amd64.changes
#...
```

##### The content of the .debian.tar.xz

As described before, this file contains the 'debian/' directory and any patch contributed by the package maintainer.

This is the content of the file for our example:

```sh
tar tf ../hello_2.12-1.debian.tar.xz
# debian/
# debian/README.Debian
# debian/changelog
# debian/control
# debian/copyright
# debian/patches/
# debian/patches/series
# debian/rules
# debian/source/
# debian/source/format
# debian/watch
```

#### Generated files in the build directory (.)

The generated files in the build directory are:

- 'debian/files' - The list of generated files. It is used while building packages to record which files are being generated. Both 'dpkg-genbuildinfo' and 'dpkg-genchanges' reads the data here when producing .buildinfo and .changes respectively. **This file is not a permanent part of the source tree and should not exist in a shipped source package**, , and so it should be removed by the 'debian/rules clean' target. See [4.12. Generated files list: debian/files in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-source.html#generated-files-list-debian-files) for details.
- 'debian/debhelper-build-stamp' - The dh's stamp file. In compat 10 (or later), dh creates a stamp file 'debian/debhelper-build-stamp' after the build step(s) are complete to avoid re-running them. It is possible to avoid the stamp file by passing '--without=build-stamp' to 'dh'. See INTERNALS section in `man dh` for details.
- 'debian/.debhelper/' - Temporal files generated by debhelper commands.
- 'debian/hello.substvars' - Debian source substitution variables. Before 'dpkg-source', 'dpkg-gencontrol' and 'dpkg-genchanges' write their control information they perform some variable substitutions on the output file.. See `man 5 deb-substvars`.
- 'debian/hello/' - The generated content of the binary package - When 'dh_auto_install' is executed, it by default will install the files inside the '<\<PKGBUILDDIR\>>/debian/<package>' directory with a command like for example `make -j4 install DESTDIR=/home/apaz/packaging-example/hello-2.12/debian/hello`.  See `man dh_auto_install` for details.
- 'debian/hello/DEBIAN/control' - Contains the most vital (and version-dependent) information about a binary package. See `man dh_gencontrol`, `man dpkg-gencontrol` and [5.3. Binary package control files – DEBIAN/control in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-controlfields.html#binary-package-control-files-debian-control) for details.
- 'debian/hello/DEBIAN/md5sums' - Lists the md5sums of each file in the package. These files are used by 'dpkg --verify' or the 'debsums' program.. See `man dh_md5sums` for details.

#### Example content in 'debian/hello.substvars'

```sh
cat debian/hello.substvars
# shlibs:Depends=libc6 (>= 2.14)
# misc:Depends=
# misc:Pre-Depends=
```

### (4.4) Fixing errors found by 'lintian'

'lintian' is a comprehensive package checker for Debian packages that tries to check for:

- Debian Policy violations and violations of various sub-policies,
- best practices,
- common mistakes,
- and problems that maintainers like to catch before uploads.

'lintian' can be run on the source .dsc metadata file, .deb binary packages or .changes file.

Example:

```sh
lintian ../hello_2.12-1.dsc
# E: hello source: malformed-contact Maintainer Aldo <>
# W: hello source: bad-homepage <insert the upstream URL, if relevant>
# W: hello source: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello source: syntax-error-in-dep5-copyright debian/copyright: Continuation line not in paragraph (line 2178). Missing a dot on the previous line?
# W: hello source: useless-autoreconf-build-depends dh-autoreconf

lintian ../hello_2.12-1_amd64.changes
# E: hello: bogus-mail-host-in-debian-changelog Aldo <>
# E: hello: changelog-is-dh_make-template
# E: hello: copyright-file-contains-full-gpl-license
# E: hello source: malformed-contact Maintainer Aldo <>
# E: hello: malformed-contact Maintainer Aldo <>
# E: hello-dbgsym: malformed-contact Maintainer Aldo <>
# E: hello changes: malformed-contact Changed-By Aldo <>
# E: hello changes: malformed-contact Maintainer Aldo <>
# E: hello: section-is-dh_make-template
# W: hello source: bad-homepage <insert the upstream URL, if relevant>
# W: hello: bad-homepage <insert the upstream URL, if relevant>
# W: hello: copyright-has-url-from-dh_make-boilerplate
# W: hello: initial-upload-closes-no-bugs
# W: hello: readme-debian-contains-debmake-template
# W: hello source: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello source: syntax-error-in-dep5-copyright debian/copyright: Continuation line not in paragraph (line 2178). Missing a dot on the previous line?
# W: hello source: useless-autoreconf-build-depends dh-autoreconf
# W: hello: wrong-bug-number-in-closes #nnnn in the installed changelog (line 3)

lintian ../hello_2.12-1_amd64.deb
# E: hello: bogus-mail-host-in-debian-changelog Aldo <>
# E: hello: changelog-is-dh_make-template
# E: hello: copyright-file-contains-full-gpl-license
# E: hello: malformed-contact Maintainer Aldo <>
# E: hello: section-is-dh_make-template
# W: hello: bad-homepage <insert the upstream URL, if relevant>
# W: hello: copyright-has-url-from-dh_make-boilerplate
# W: hello: initial-upload-closes-no-bugs
# W: hello: readme-debian-contains-debmake-template
# W: hello: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# W: hello: wrong-bug-number-in-closes #nnnn in the installed changelog (line 3)
```

NOTE: To see verbose description of the problems we can use the '--info' flag or the 'lintian-info' command. For Python packages, there is also a 'lintian4python' tool that provides some additional lintian checks.

Reference: [Lintian User's Manual](https://lintian.debian.org/manual/index.html).

In the next subsections we sill be fixing errors (E) and warnings (W) for our example and running 'debuild' with the '--no-pre-clean' (or the shortcut '-nc') option of 'dpkg-buildpackage' to rebuild the package without compiling again:

```sh
# ... changes
debuild -nc
# dpkg-buildpackage -us -uc -ui -nc
lintian
```

After applying the changes in the subsections below, 'lintian' will pass:

```sh
debuild
# ...
# Now running lintian hello_2.12-1_amd64.changes ...
# W: hello: initial-upload-closes-no-bugs
# Finished running lintian.

echo $?
#0

lintian ../hello_2.12-1.dsc

lintian ../hello_2.12-1_amd64.changes
# W: hello: initial-upload-closes-no-bugs

lintian ../hello_2.12-1_amd64.deb
# W: hello: initial-upload-closes-no-bugs
```

#### Issues at 'debian/README.Debian'

```
W: hello: readme-debian-contains-debmake-template
# details: https://lintian.debian.org/tags/readme-debian-contains-debmake-template
```

NOTE: this file could be deleted if you think no notes regarding this package are needed.

Changed 'debian/README.Debian' from:

```
hello for Debian

Please edit this to provide information specific to
this hello Debian package.

    (Automatically generated by debmake Version 4.3.2)

 -- Aldo <>  Thu, 19 May 2022 01:38:47 -0300
```

to:

```
GNU hello for Debian
=====================

The GNU Hello program produces a familiar, friendly greeting. Yes, this is another implementation of the classic program that prints "Hello, world!" when you run it.

However, unlike the minimal version often seen, GNU Hello processes its argument list to modify its behavior, supports greetings in many languages, and so on. The primary purpose of GNU Hello is to demonstrate how to write other programs that do these things; it serves as a model for GNU coding standards and GNU maintainer practices.

GNU Hello is written in C. For implementations in other programming languages, notably including translation into other languages, please see the GNU Gettext distribution.

 -- Aldo Paz <aldo.paz@noemail.org>  Thu, 19 May 2022 01:38:47 -0300
```

#### Issues at 'debian/control'

Here we will fix these issues:

```
E: hello source: malformed-contact Maintainer Aldo <>
E: hello: malformed-contact Maintainer Aldo <>
E: hello-dbgsym: malformed-contact Maintainer Aldo <>
E: hello changes: malformed-contact Maintainer Aldo <>
E: hello changes: malformed-contact Changed-By Aldo <>
# details: https://lintian.debian.org/tags/malformed-contact

E: hello: section-is-dh_make-template
# details: https://lintian.debian.org/tags/section-is-dh_make-template

W: hello: bad-homepage <insert the upstream URL, if relevant>
# details: https://lintian.debian.org/tags/bad-homepage

W: hello: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
# details: https://lintian.debian.org/tags/superfluous-clutter-in-homepage

W: hello source: useless-autoreconf-build-depends dh-autoreconf
# details: https://lintian.debian.org/tags/useless-autoreconf-build-depends
```

Changed 'debian/control' from:

```
Source: hello
Section: unknown
Priority: optional
Maintainer: Aldo <>
Build-Depends: debhelper-compat (= 12), dh-autoreconf
Standards-Version: 4.5.0
Homepage: <insert the upstream URL, if relevant>

Package: hello
Architecture: any
Multi-Arch: foreign
Depends: ${misc:Depends}, ${shlibs:Depends}
Description: auto-generated package by debmake
 This Debian binary package was auto-generated by the
 debmake(1) command provided by the debmake package.
```

to:

```
Source: hello
Section: devel
Priority: optional
Maintainer: Aldo Paz <aldo.paz@noemail.org>
Build-Depends: debhelper-compat (= 12)
Standards-Version: 4.5.0
Homepage: http://www.gnu.org/software/hello/

Package: hello
Architecture: any
Multi-Arch: foreign
Depends: ${misc:Depends}, ${shlibs:Depends}
Description: auto-generated package by debmake
 This Debian binary package was auto-generated by the
 debmake(1) command provided by the debmake package.
```

NOTE: Since compatibility level 10, 'debhelper' enables the autoreconf sequence by default. It is therefore not necessary to specify build-dependencies on 'dh-autoreconf' or 'autotools-dev' and they can be removed.

NOTE: The extended description (the lines after the first line of the "Description:" field) is required and must not be empty. See [extended-description-is-empty](https://lintian.debian.org/tags/extended-description-is-empty) and [3.4. The description of a package in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-binary.html#the-description-of-a-package) for details.

#### Issues at 'debian/changelog'

Here we will fix these issues:

```
E: hello: bogus-mail-host-in-debian-changelog Aldo <>
# details: https://lintian.debian.org/tags/copyright-file-contains-full-gpl-license

E: hello: changelog-is-dh_make-template
# details: https://lintian.debian.org/tags/changelog-is-dh_make-template

W: hello: wrong-bug-number-in-closes #nnnn in the installed changelog (line 3)
# details: https://lintian.debian.org/tags/wrong-bug-number-in-closes
```

Changed 'debian/changelog' from:

```
hello (2.12-1) UNRELEASED; urgency=low

  * Initial release. Closes: #nnnn
    <nnnn is the bug number of your ITP>

 -- Aldo <>  Thu, 19 May 2022 01:38:47 -0300
```

to:

```
hello (2.12-1) UNRELEASED; urgency=low

  * Initial packaging.

 -- Aldo Paz <aldo.paz@noemail.org>  Thu, 19 May 2022 01:38:47 -0300
```

See `man 5 deb-changelog` and `man dpkg-parsechangelog` for format and details.

##### The distribution value in a changelog entry is important

In order to prevent a package being accidentally uploaded before completing the package, it is a good idea to change the distribution value to an invalid distribution value of UNRELEASED.

Once you are satisfied with all the changes and documented them in changelog, you should change the distribution value from UNRELEASED to the target distribution value unstable (or even experimental).

##### Ignoring lintian's 'initial-upload-closes-no-bugs' tag

```
W: hello: initial-upload-closes-no-bugs
# defails: https://lintian.debian.org/tags/initial-upload-closes-no-bugs
```

The details about this warning says:

```
This package appears to be the first packaging of a new upstream software package (there is only one changelog entry and the Debian revision is 1), but it does not close any bugs. The initial upload of a new package should close the corresponding ITP bug for that package.

This warning can be ignored if the package is not intended for Debian or if it is a split of an existing Debian package.
```

Because we don't plan to upload this new package to debian, we will ignore this warning.

See [5.1. New packages in the Debian Developer's Reference](https://www.debian.org/doc/manuals/developers-reference/pkgs.html#new-packages) for details.

##### Modifying changelog entries with 'debchange' (dch)

- `dch -e` - Open 'debian/chengelog' with default editor.
- `dch -a "Summary of a change"` - Add a new changelog entry to the current version of the package.
- `dch -i "Summary of a change"` - Increment the version of the package (e.g. 0.13.0-0ubuntu5 => 0.13.0-0ubuntu6) adding a new release entry, and add a new changelog entry.
- `dch -r ""` - Finalize the changelog for a release. Update the name, email, timestamp, and distribution of the last release entry.
- `dch -r --distribution <CODENAME>` - Similar to `dch -r ""`, finalize the changelog for a release. If the distribution is set to UNRELEASED, change it to the distribution from the previous changelog entry (or another distribution as specified by --distribution). If there are no previous changelog entries and an explicit distribution has not been specified, unstable will be used.

See `man dch` and [Chapter 8. Updating the package in the Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/update.en.html) for details.

#### Issues at 'debian/copyright'

Here we will fix these issues:

```
E: hello: copyright-file-contains-full-gpl-license
# details: https://lintian.debian.org/tags/copyright-file-contains-full-gpl-license

W: hello: copyright-has-url-from-dh_make-boilerplate
# details: https://lintian.debian.org/tags/copyright-has-url-from-dh_make-boilerplate

W: hello source: syntax-error-in-dep5-copyright debian/copyright: Continuation line not in paragraph (line 2178). Missing a dot on the previous line?
# details: https://lintian.debian.org/tags/syntax-error-in-dep5-copyright
```

'debian/copyright' needs to be filled in to follow the licence of the upstream source. Fixing a 'debian/copyright' file implies extracting all the different copyright licenses in the upstream's files.

'debmake' generated a copyright template including all the licenses extracted from the sources so we have to modify it to just include a simplified list with references to the corresponding complete license file in 'usr/share/common-licenses/'.

New version of 'debian/copyright':

```
This is the Debian GNU prepackaged version of the FSF's GNU hello
utility. This package is an example of how to package a GNU program.

This package was put together by Aldo Paz and it's currently unmaintained.

Support was added for the Debian package maintenance scheme, by
including various debian/* files.

Program Copyright 1992-2020 Free Software Foundation, Inc.

Modifications for Debian Copyright (C) 2022 Aldo Paz.

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.
.
This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
.
You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
.
On Debian systems, the complete text of the GNU General Public License
Version 3 can be found in `/usr/share/common-licenses/GPL-3'.

On Debian systems, the complete text of the GNU General Public License
can be found in `/usr/share/common-licenses/GPL'.

Manual Copyright (C) 1992-2022 Free Software Foundation, Inc.

Permission is granted to copy, distribute and/or modify this
document under the terms of the GNU Free Documentation License,
Version 1.3 or any later version published by the Free Software
Foundation; with no Invariant Sections, with no Front-Cover Texts,
and with no Back-Cover Texts.  A copy of the license is included in
the section entitled "GNU Free Documentation License".

On Debian systems, the complete text of the GNU Free Documentation
License can be found in `/usr/share/common-licenses/GFDL'.
```

See [12.5. Copyright information in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-docs.html#copyright-information) and [DEP 5: Machine-readable debian/copyright](https://dep-team.pages.debian.net/deps/dep5) for details.

##### License checkers

Source files can be analized to check the license they have. One useful command is 'licensecheck' that attempts to determine the license that applies to each file passed to it, by searching the start of the file for text belonging to various licenses.

Other similar tools exist. Here is a list of known tools also command-line based and general-purpose:
- copyright-update <https://github.com/jaalto/project--copyright-update> - Written in Perl.
- debmake <http://anonscm.debian.org/git/collab-maint/debmake.git> - Written in Python, specific to Debian packages. It has several options for license analysis and management. See `man debmake` for details.
- decopy <https://anonscm.debian.org/git/collab-maint/decopy.git> - Written in Python. Item Licensee <http://ben.balter.com/licensee/> written in Ruby.
- LicenseFinder <https://github.com/pivotal/LicenseFinder> - Written in Ruby.
- ninka <http://ninka.turingmachine.org/> - Written in C++. Used in FOSSology <http://fossology.org/> (along with Monk and Nomos apparently unavailable as standalone command-line tools).
- ripper <https://github.com/odeke-em/ripper> - Written in Go.
- scancode-toolkit <https://github.com/nexB/scancode-toolkit> - Written in Python.

See `man 1p licensecheck`, `man debmake` and [Debian wiki: CopyrightReviewTools](https://wiki.debian.org/CopyrightReviewTools) for details.

## (5) Package signing

**Signing data with a GPG key enables the recipient of the data to verify that no modifications occurred after the data was signed (assuming the recipient has a copy of the sender’s public GPG key)**.

Debian package files (.deb files), Debian source packages (.dsc files), and Debian changes files (.changes files) can all be signed with GPG.

Many Debian-based Linux distributions (e.g., Ubuntu) have GPG signature verification of Debian package files (.deb) disabled by default and instead **choose to verify GPG signatures of repository metadata and source packages (.dsc)**. The setting which enables GPG signature checking of the individual .deb packages can be found in /etc/dpkg/dpkg.cfg and is set to 'no-debsig', but there are important caveats to enabling this option.

Further, most official Debian package files from the publicly accessible repositories do not have GPG signatures. **The official repository metadata is GPG signed, as are the source packages, but the .deb packages themselves are not**.

If you publish a Debian package and GPG sign the package yourself before distributing it to users, those users’ systems will, in most cases, not verify the signature of the package unless they have done a considerable amount of configuration. However, **their system will, in most cases, automatically verify repository metadata**.

See [7.5. Package signing in Debian in the Securing Debian Manual](https://www.debian.org/doc/manuals/securing-debian-manual/deb-pack-sign.en.html) for details.

### 'debuild' and package signing

After calling 'lintian', 'debuild' signs the .changes and/or .dsc files as appropriate (using 'debsign' to do this instead of 'dpkg-buildpackage' itself; all relevant key-signing options are passed on). Signing occurs automatically if you have a primary key in you gnupg keychain, and you run 'dpkg-buildpackage' or 'debuild' without the '-us' and '-uc' switches.

If there isn't a primary key in your gnupg keychain, then 'debuild' by default executes 'dpkg-buildpackage' with '-us' and '-uc', that disables signing the package with GPG.

### Creating a GPG key

GPG stands for GNU Privacy Guard and it implements the OpenPGP standard which allows you to sign and encrypt messages and files. This is useful for a number of purposes. In our case it is important that you can sign files with your key so they can be identified as something that you worked on.

```sh
gpg --quick-generate-key "Aldo Paz <aldo.paz@noemail.org>" rsa4096 default never
# ...

gpg --list-keys --keyid-format short
# /home/apaz/.gnupg/pubring.kbx
# -----------------------------
# pub   rsa4096/17758171 2022-05-24 [SC]
#       0837A887276EF500C9F3FC98AADB3BFE17758171
# uid         [ultimate] Aldo Paz <aldo.paz@noemail.org>
```

GPG will ask for a passphrase (a passphrase is just a password which is allowed to include spaces), choose a safe one.

NOTE: Before GPG creates a key for you, it needs random bytes. So if you give the system some work to do it will be just fine; move the cursor around, type some paragraphs of random text, load some web page.

In this case 17758171 is the key ID.

#### Create a backup of your GPG keys

GPG keys, secret keys, subkeys and ownertrust can be exported and imported again with following commands:

```sh
# Export
gpg --export --armor "Aldo Paz" > mykey.pub.asc
gpg --export-secret-keys --armor "Aldo Paz" > mykey.priv.asc
gpg --export-secret-subkeys --armor "Aldo Paz" > mykey.sub_priv.asc
gpg --export-ownertrust > ownertrust.txt

# Import
gpg --import mykey.pub.asc
gpg --import mykey.priv.asc
gpg --import mykey.sub_priv.asc
gpg --import-ownertrust ownertrust.txt
```

### GPG signing Debian source packages (.dsc) and changes (.changes) files

The Debian source package file (.dsc) and changes file (.changes) are text-based files and both contain SHA1, SHA256, and MD5 checksums of all source files that comprise the source package. Since this data is signed, any modifications to the source file checksums, file size, or other data in the dsc or changes files would cause signature verification to fail.

In case of .dsc files, as long as a recipient who wishes to build the .deb package from a .dsc file, he verifies the checksums of each file in the source package and verifies the signature of the source package itself, the recipient knows their build uses precisely the same files that the packager used when building.

Theses files can be manually signed with a GPG key by using 'debsign'. 'debsign' embeds the signature in the file and it can be verified using 'gpg'.

Example:

```sh
# sign using GPG key ID: E732A79A
debsign -k E732A79A ../hello_2.12-1.dsc
debsign -k E732A79A ../hello_2.12-1_amd64.changes


# verify
gpg --verify ../hello_2.12-1.dsc
gpg --verify ../hello_2.12-1_amd64.changes
```

`apt-get source <packagename>` will automatically check the GPG signature of the source package file and the checksums of the source files listed in the source package (.dsc).

See `man debsign` for details.

### GPG signing Debian binary packages (.deb) files

A Debian binary package file (.deb) itself is actually an AR archive. This file can be signed with a GPG key using 'debsigs' (not debsign). The signature is included in the archive and is named based on its role.

Some commonly used roles include:
- origin: the signature of the organization which distributes the .deb file.
- maint: the signature of the package maintainer.
- archive: the signature of the archive providing the package to reassure users that the package is the actually provided by the archive.

Example:

```sh
# sign using GPG key ID: E732A79A
debsigs --sign=origin -k E732A79A ../hello_2.12-1_amd64.deb
```

Thus the .deb package will be modified to include a file called '_gpgorigin' that contains the GPG signature of both the 'debian-binary' file, 'control' archive, and 'data' archive (files inside this .deb archive) concatenated together.

Verifying signatures on .deb package files is a bit more involved than the other verification methods.

See `man 1p debsigs` for details.

#### Verifying GPG signatures of .deb package files

In order to verify .deb package files, you must have the program 'debsig-verify' installed, import the public GPG keys you will use to verify packages to the 'debsig-verify' keyrings, and you must also create an XML policy document for signature verification.

(1) create a keyring directory for the public key and import the public key to the 'debsig-verify' GPG keyring:

```sh
mkdir /usr/share/debsig/keyrings/DDDF2F4CE732A79A/
gpg --no-default-keyring \
    --keyring /usr/share/debsig/keyrings/DDDF2F4CE732A79A/debsig.gpg \
    --import my-public-key
```

In the above example, the keyring subdirectory, DDDF2F4CE732A79A, is the fingerprint of the GPG key which will be imported. Note that the name of the keyring 'debsig.gpg' will be used in the XML policy document created next. You can use any filename you like.

(2) create a directory for the policy document. It must have the GPG key fingerprint in the path:

```sh
mkdir /etc/debsig/policies/DDDF2F4CE732A79A/
```

(3) create the XML document. The 'debsig-verify' man page indicates that the filename is irrelevant so long as the filename ends with .pol and resides in a directory named after the GPG key fingerpint. See '/usr/share/doc/debsig-verify/policy-syntax.txt' and '/usr/share/doc/debsig-verify/examples' for details.

Example:

```xml
<?xml version="1.0"?>
<!DOCTYPE Policy SYSTEM "http://www.debian.org/debsig/1.0/policy.dtd">
<Policy xmlns="http://www.debian.org/debsig/1.0/">

  <Origin Name="test" id="DDDF2F4CE732A79A" Description="Test package"/>

  <Selection>
    <Required Type="origin" File="debsig.gpg" id="DDDF2F4CE732A79A"/>
  </Selection>

   <Verification MinOptional="0">
    <Required Type="origin" File="debsig.gpg" id="DDDF2F4CE732A79A"/>
   </Verification>
</Policy>
```

(4) Now you can verify the GPG signature of the Debian .deb package:

Example:

```sh
debsig-verify test_1.0-7_amd64.deb
# debsig: Verified package from `Test package' (test)
```

#### dpkg support for 'debsig-verify'

dpkg has support for verifying GPG signatures of Debian package files, but this verification is disabled by default. This means that when a package is installed on a Debian-based system, the signature checking for individual packages is disabled.

In order to enable it, the file '/etc/dpkg/dpkg.cfg' will need to be modified to remove the 'no-debsig' option from the config file.

CAUTION: Doing this may cause dpkg to reject installation of unsigned Debian files. If you plan to enable this, it is strongly recommended you try this first in a testing environment. You’ll also need to ensure you have the correct XML policy files created, as explained above.

#### 'dpkg-sig', another .deb file signing tool

There is a another tool available for signing and verifying Debian package files called 'dpkg-sig'. Signatures created by this tool are not compatible with debsigs and the tools cannot understand or verify each other signatures.

'dpkg-sig' generates a Debian control file with several userful fields:
- Version of dpkg-sig which generated the signature
- Gpg key signer information
- Role
- Files section with checksums, file sizes, and filenames of the control, data, and debian-binary files in the Debian package

This file is named based on the role selected, which is defaulted to builder if none is specified. So, signing a Debian package file with no role would result in a file named '_gpgbuilder' being created and added to the package file. This file is signed with GPG and the signature is embedded as plain text (a "clearsign" GPG signature).

The result of this scheme is that the GPG signature can be verified using commonly available software. The Debian package can be extracted with 'ar' and the signature of the control file can be checked with just `gpg --verify`.

Signing a Debian file with 'dpkg-sig' is straight forward:

```sh
dpkg-sig -k E732A79A --sign builder ../hello_2.12-1_amd64.deb
```

The signature can also be checked by using dpkg-sig directly, if desired:

```sh
dpkg-sig --verify test_1.0-7_amd64.deb
# Processing test_1.0-7_amd64.deb...
# GOODSIG _gpgbuilder 284E8BE753AE45DFF8D82748DDDF2F4CE732A79A 1414371553
```

Additionally, no XML policy document is required for signature verification, which makes signature verification much simpler to integrate into a workflow.

#### dpkg support for dpkg-sig

Unfortunately, dpkg does not have built-in support for 'dpkg-sig', but the discussion appears to have been moved to a bug ticket with Debian: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=340306.

### GPG signatures for APT repository metadata

It is also possible to generate GPG signatures for APT repositories in addition to generating signatures for Debian packages themselves.

Generating signatures of the repository metadata allows you to verify that the repository metadata has not been tampered with since it was generated by the repository host.

This is critical to preventing a Man-in-the-Middle attack attack where a malicious actor plants repository metadata causing a user’s package manager to install potentially malicious packages.

Luckily, generating GPG sigantures for APT repositories is straightforward with 'reprepro'.

Assuming you have a GPG key you’d like to use to sign the repository metadata, you can simply modify your reprepro repository config to add your GPG key.

Using the same GPG key ID used in the earlier examples, the 'conf/distributions' config file can be modified to add the field:

```
SignWith: E732A79A
```

This will cause 'reprepro' to generate GPG signatures of the repository metadata. 'reprepro' will generate a signature of the apt 'Release' file and store the signature in the file 'Release.gpg'. This is called a "detached GPG signature."

Newer version of 'reprepro' will generate an embedded text GPG signature (a "clearsign" GPG signature) in a file named 'InRelease'.

The 'apt' client will download the GPG Release file(s) it understands and will automatically attempt to verify the GPG signature of the repository.

Users of the repository will need to have the GPG key installed on the system. This can be done by end users running 'apt-key':

```sh
sudo bash -c 'wget -O - https://packagecloud.io/gpg.key | apt-key add -'
```

The above command downloads a GPG key and adds the GPG key to the system using 'apt-key'.

## (6) Functional tests with DEP-8

'automatic as-installed package tests' can be picked up by an infrastructure like ci.debian.net. Also 'lintian' is now encouraging maintainers to add 'autopkgtest' metadata.

There are two tools that allows implementing DEP-8:
- 'sadt' - simple DEP-8 test runner. See `man sadt` for details.
- 'autopkgtest' - test an installed binary package using the source package's tests. See `man autopkgtest` for details.

See [DEP 8: automatic as-installed package testing](https://dep-team.pages.debian.net/deps/dep8/) for details.

### Creating a simple automated as-installed test

'debian/tests/control':

```
Test-Command: hello
Depends: hello

Tests: it-prints-the-message
Depends: @
```

'debian/tests/it-prints-the-message':

```
#!/bin/sh

set -e

message=`LC_ALL=C hello`
echo "This should be a friendly greeting: '$message'"
test "$message" = "Hello, world!"
```

### Running the tests

Inside the build directory we could run one of the following commands for executing the tests:

```sh
# option 1
autopkgtest -B . -- null

# option 2
sadt --verbose
```

## Files in 'debian/' directory

See [Chapter 5. Other files under the debian directory in the Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/dother.en.html) for details.

### The minimum required for packaging

The minimum required files are:

```
debian/source/format
debian/control
debian/changelog
debian/copyright
debian/rules
```

### 'debian/compat'

It is not required enymore. In current 'debhelper' versions, you can specify the compatibility level in 'debian/control' by adding a Build-Depends on the 'debhelper-compat' package.

Prior versions of 'debhelper' required specifying the compatibility level in the file 'debian/compat', and current 'debhelper' still supports this for backward compatibility. To use this method, the debian/compat file should contain the compatibility level as a single number, and no other content. If you specify the compatibility level by this method, your package will also need a versioned build dependency on a version of the debhelper package equal to (or greater than) the compatibility level your package uses.

So, if you specify compatibility level 13 in 'debian/compat', ensure 'debian/control' has:

```
Build-Depends: debhelper (>= 13~)
```

Note that you must use either the build-dependency on 'debhelper-compat' or the 'debian/compat' file. Whenever possible, the 'debhelper-compat' build-dependency is recommended.

See `man 7 debhelper` for details.

### 'debian/README.source' and 'debian/README.Debian'

'debian/README.source' and 'debian/README.Debian' are only needed if your package has any non-standard features.

'debian/README.Debian' should document any extra details or discrepancies between the original package and the Debian version.

'debian/README.source' (not 'debian/README.Debian') may include any information that would be helpful to someone modifying the source package. Maintainers are encouraged to document in a 'debian/README.source' file any source package with a particularly complex or unintuitive source layout or build system (for example, a package that builds the same source multiple times to generate different binary packages). See [4.14. Source package handling: debian/README.source in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-source.html#source-package-handling-debian-readme-source) for details.

### 'debian/*.docs'

Any 'debian/*.docs' file list documentation files to be installed into package. Supports substitution variables in compat 13 and later as documented in 'debhelper'. See `man dh_installdocs` for details.

### 'debian/watch'

TODO

## What are the debhelper commands that 'dh' will execute for a given sequence?

It could be confusing to document which commands are run, in which order, for all the possible build targets and configuration arrangements (or even just for the most common ones).

The sequence of helper tools that will get executed depends on a few things:

1. what build target is being passed. These include: 'build-arch', 'build-indep', 'build', 'clean', 'install-indep', 'install-arch', 'install', 'binary-arch', 'binary-indep', and 'binary'. The meanings of (most of) these are discussed in the [4.9. Main building script: debian/rules in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-source.html#main-building-script-debian-rules) page.
2. the Debhelper compat level (as found in the 'debian/compat' file).
3. your version of Debhelper (although an effort is made to make different versions work the same given the same compat level).
4. what helper commands have already been run since the last clean (in debhelper compat levels 9 and lower).
5. what addons are being used (the '--with' and '--without' options).
6. what override targets exist in the makefile (e.g. override_dh_auto_test).

The way to know, therefore, is to use the '--no-act' argument to 'dh', with the build directory set up the way you want it. '--no-act' prints commands that would run for a given sequence, but does not run them.

NOTE: 'dh' normally skips running commands that it knows will do nothing.  With '--no-act', the full list of commands in a sequence is printed.

Example, 'clean' sequence for autotools based project:

```sh
# Default clean
dh clean --no-act
#    dh_testdir
#    dh_auto_clean
#    dh_autoreconf_clean
#    dh_clean


# Using 'quilt' addon
dh clean --with quilt --no-act
#    dh_testdir
#    dh_auto_clean
#    dh_autoreconf_clean
#    dh_quilt_unpatch
#    dh_clean
```

## Skipping signing

Normally 'dpkg-buildpackage' runs the 'sign' hook and calls 'gpg2' or 'gpg' to sign the following files as long as it is not an UNRELEASED build:
- '*.dsc': the Debian source packages' control file. See `man 5 dsc` for details.
- '*.buildinfo': the Debian build information file. Each Debian source package build can record the build information in a .buildinfo control file. The name of the .buildinfo file will depend on the type of build and will be as specific as necessary but not more; for a build that includes **any** the name will be source-name_binary-version_arch.buildinfo, or otherwise for a build that includes **all** the name will be source-name_binary-version_all.buildinfo, or otherwise for a build that includes **source** the name will be source-name_source-version_source.buildinfo. See `man 5 deb-buildinfo` and `man dpkg-genbuildinfo` for details.
- '*.changes': the Debian changes file, an Debian upload control file. See `man 5 deb-changes` and `man dpkg-genchanges` for details.

Signing can be skipped with ont of the following methods:

Method 1, running `dpkg-buildpackage -us -uc -ui` where:

```
-us, --unsigned-source
              Do not sign the source package
-uc, --unsigned-changes
              Do not sign the .buildinfo and .changes files
-ui, --unsigned-buildinfo
              Do not sign the .buildinfo file
```

Method 2, running `dpkg-buildpackage --no-sign` that do not sign any file, this includes the source package, the '.buildinfo' file and the '.changes' file.

## Each source package may generate several binary packages

It is important to note here that there is no required correspondence between the name of the source package and that of the binary package(s) that it generates. It is easy enough to understand if you know that each source package may generate several binary packages. This is why the .dsc file has the Source and Binary fields to explicitly name the source package and store the list of binary packages that it generates.

Quite frequently, a source package (for a given software) can generate several binary packages. The split is justified by the possibility to use (parts of) the software in different contexts. Consider a shared library, it may be installed to make an application work (for example, libc6), or it can be installed to develop a new program (libc6-dev will then be the correct package). We find the same logic for client/server services where we want to install the server part on one machine and the client part on others (this is the case, for example, of openssh-server and openssh-client).

Just as frequently, the documentation is provided in a dedicated package: the user may install it independently from the software, and may at any time choose to remove it to save disk space. Additionally, this also saves disk space on the Debian mirrors, since the documentation package will be shared amongst all of the architectures (instead of having the documentation duplicated in the packages for each architecture).

See [5.3. Structure of a Source Package in the The Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/sect.source-package-structure.html) for details.

## Each modification in a Debian package is the consequence of a modification made to the source package

The source package is the foundation of everything in Debian. All Debian packages come from a source package, and each modification in a Debian package is the consequence of a modification made to the source package. The Debian maintainers work with the source package, knowing, however, the consequences of their actions on the binary packages. The fruits of their labors are thus found in the source packages available from Debian: you can easily go back to them and everything stems from them.

When a new version of a package (source package and one or more binary packages) arrives on the Debian server, the source package is the most important. Indeed, it will then be used by buildd - a network of machines of different architectures for compilation on the various architectures supported by Debian. The fact that the developer also sends one or more binary packages for a given architecture (usually i386 or amd64) is relatively unimportant, since these could just as well have been automatically generated.

See [5.3. Structure of a Source Package in the The Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/sect.source-package-structure.html) for details.

## Source only maintainer uploads since Debian 10 Buster

Right after the release of Debian 10 Buster the Release Team announced that maintainer binary uploads will no longer be accepted for main and all binary packages in this component will be built automatically from mandatory source-only uploads.

See [5.3. Structure of a Source Package in the The Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/sect.source-package-structure.html) for details.

## The build logs in buildd

The logs of builds of binary packages on buildd (the Debian Package Auto-Building system) are public in [https://buildd.debian.org/](https://buildd.debian.org/). For example [this is the log of hello-2.10 at 2019-05-13 18:31:23](https://buildd.debian.org/status/fetch.php?pkg=hello&arch=amd64&ver=2.10-2&stamp=1557772283&raw=0).

## The 'wrap-and-sort' command

'wrap-and-sort' is an script part of the 'devscripts' package that wraps the package lists in Debian control files. By default the lists will only split into multiple lines if the entries are longer than the maximum line length limit of 79 characters.

I also sorts the package lists in Debian control files and all .dirs, .docs, .examples, .info, .install, .links, .maintscript, and .manpages files. Beside that wrap-and-sort removes trailing spaces in these files.

This script should be run in the root of a Debian package tree. It searches for control, control*.in, copyright, copyright.in, install, and *.install in the debian directory.

See `man wrap-and-sort` for details.

## The 'quilt' command

Quilt  is a tool to manage large sets of patches by keeping track of the changes each patch makes.

See `man quilt` and [UsingQuilt](https://wiki.debian.org/UsingQuilt) for details.

## Alternatives for binary package building

### Low level tools

- 'debian/rules': maintainer script for the package building, this file defines how the Debian binary package is built.
- 'dpkg-buildpackage': core of the package building tool, the official command to build the Debian binary package.

### High level tools

NOTE: Although use of higher level commands such as 'gbp buildpackage' and 'pbuilder' ensures the perfect package building environment, it is essential to understand how lower level commands such as 'debian/rules' and 'dpkg-buildpackage' are executed under them.

- 'debuild': dpkg-buildpackage + lintian (build under the sanitized environment variables), the most popular tool. A wrapper script of the dpkg-buildpackage command to build the Debian binary package under the proper environment variables.
- 'pbuilder': core of the Debian chroot environment tool.
- 'pdebuild': pbuilder + dpkg-buildpackage (build in the chroot).
- 'cowbuilder': speed up the pbuilder execution.
- 'git-pbuilder: the easy-to-use commandline syntax for pdebuild (used by gbp buildpackage).
- 'gbp': part of the git-buildpackage (a suite to help with Debian packages in Git repositories), manage the Debian source under the git repo.
- 'gbp buildpackage': pbuilder + dpkg-buildpackage + gbp.
- 'sbuild': a wrapper script to build the Debian binary package under the proper chroot environment with the proper environment variables. Unlike under the pbuilder package, the chroot environment under the sbuild package used by the autobuilder system does not enforce the use of a minimal system and may have many leftover packages installed.

See [Chapter 6. Building the package in the Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/build.en.html) for details.

## Debian packages - native vs non-native

A native Debian package has the "3.0 (native)" format.

A non-native Debian package has the "3.0 (quilt)" format. It is more friendly to the downstream distributions.

## The 'git-buildpackage' package

'git-buildpackage' is a suite to help with maintaining Debian packages in Git repositories.

TODO

See [git-buildpackage documentation](https://honk.sigxcpu.org/piki/projects/git-buildpackage/) and [PackagingWithGit in the Debian wiki](https://wiki.debian.org/PackagingWithGit) for details.

## Reference

- [Debian's packaging portal](https://wiki.debian.org/Packaging) - It is the wiki that gathers everything about packaging
- [Ubuntu Packaging Guide](https://packaging.ubuntu.com/html/)
- (PDF) [Introduction to Debian packaging](https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.en.pdf) - Package: packaging-tutorial
- [Guide for Debian Maintainers](https://www.debian.org/doc/manuals/debmake-doc/index.en.html) - Read [Chapter 4. Simple Example](https://www.debian.org/doc/manuals/debmake-doc/ch04.en.html) for an example of creating a simple Debian package from a simple C source using the Makefile as its build system, and [Chapter 8. More Examples](https://www.debian.org/doc/manuals/debmake-doc/ch08.en.html) for examples with many upstream cases.
- (Outdated) [Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/index.en.html)
- [Debian Policy Manual](https://www.debian.org/doc/debian-policy/)
- [Debian Developer's Reference](https://www.debian.org/doc/manuals/developers-reference/)
- [The Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/index.en.html)
- [Debian Developers' Corner](https://www.debian.org/devel/)
- [Debian Developers' Manuals](https://www.debian.org/doc/devel-manuals)
- [DEP - Debian Enhancement Proposals](https://dep-team.pages.debian.net/)
