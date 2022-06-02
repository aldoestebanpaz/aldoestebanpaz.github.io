# Creating packages for Ubuntu

## (1) Create and configure your account on Launchpad

Launchpad is the Ubuntu's hosting platform (besides hosting Ubuntu, Launchpad can host any Free Software project). It is the central piece of infrastructure in Ubuntu and stores:
- packages,
- code,
- translations,
- bug reports,
- proposed fixes (to get other Ubuntu developers to review and sponsor them),
- information about the people who work on Ubuntu and their team memberships.

We will need to register with Launchpad and provide a minimal amount of information to be able to create and upload anything.

For more information see the [Launchpad Help wiki](https://help.launchpad.net/).

### (1.1) Get a Launchpad account and account ID

If you don't already have a Launchpad account, create one going to [https://launchpad.net/+login](https://launchpad.net/+login).

NOTE: Launchpad's registration process will ask you to choose a display name. You should use your real name here so that your Ubuntu developer colleagues will be able to get to know you better.

We can find our Launchpad ID by going to [https://launchpad.net/~](https://launchpad.net/~) or [https://launchpad.net/people/+me/](https://launchpad.net/people/+me/) and looking for the part after the '~' in the URL. Mine is 'apaz'.

See [YourAccount/NewAccount](https://help.launchpad.net/YourAccount/NewAccount) for other settings configuration.

## (2) Install packaging-related software

We need to install essential packaging-related software. This includes:
- Ubuntu-specific packaging utilities
- Encryption software so your work can be verified as being done by you
- Additional encryption software so you can securely transfer files

### (2.1) Install the essential packages

```sh
sudo apt install gnupg pbuilder ubuntu-dev-tools apt-file
```

It installs:
- 'gnupg' - GNU Privacy Guard contains tools you will need to create a cryptographic key with which you will sign files you want to upload to Launchpad.
- 'pbuilder' - a tool to do reproducible builds of a package in a clean and isolated environment.
- 'ubuntu-dev-tools' (and 'devscripts', a direct dependency) - a collection of tools that make many packaging tasks easier.
- 'apt-file' - provides an easy way to find the binary package that contains a given file.

### (2.2) Export configuration variables for these tools

The Debian/Ubuntu packaging tools will use the following environment variables to replace put your information in where necessary (for example, in the changelog).

Append 'DEBFULLNAME' and 'DEBEMAIL' to  '~/.bashrc' and either restart your terminal or source this file:

```sh
echo 'export DEBFULLNAME="Aldo Paz"' >> ~/.bashrc
echo 'export DEBEMAIL="aldo.paz@noemail.org"' >> ~/.bashrc

source ~/.bashrc
```

## (3) Create a GPG key and upload it to your Launchpad account

In some part of the process (when uploading a source package to Launchpad) you need to sign some files with a GPG key so they can be identified as something that you worked on.

GPG stands for GNU Privacy Guard and it implements the OpenPGP standard which allows you to sign and encrypt messages and files.

Find more information at [https://help.launchpad.net/YourAccount/ImportingYourPGPKey](https://help.launchpad.net/YourAccount/ImportingYourPGPKey).

### (3.1) Create a GPG key

```sh
# gpg --quick-generate-key "User Name (comments) <your@email.com>" rsa4096 default never
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

### (3.2) Upload the public part of your GPG key to the Ubuntu keyserver

You need to upload the public part of your key to a keyserver so the world can identify messages and files as yours.

```sh
# gpg --send-keys --keyserver keyserver.ubuntu.com <KEY ID>
gpg --send-keys --keyserver keyserver.ubuntu.com 17758171
```

This will send your key to the Ubuntu keyserver, but a network of keyservers will automatically sync the key between themselves. Once this syncing is complete, your signed public key will be ready to verify your contributions around the world.

### (3.3) Upload your GPG key to Launchpad

First, you will need to get your fingerprint and key ID.

```sh
gpg --fingerprint "Aldo Paz"
# pub   rsa4096 2022-05-24 [SC]
#       0837 A887 276E F500 C9F3  FC98 AADB 3BFE 1775 8171
# uid           [ultimate] Aldo Paz <aldo.paz@noemail.org>
```

Head to [https://launchpad.net/~/+editpgpkeys](https://launchpad.net/~/+editpgpkeys) import the key fingerprint. In the case above this would be '0837 A887 276E F500 C9F3  FC98 AADB 3BFE 1775 8171'.

Launchpad will use the fingerprint to check the Ubuntu key server for your key and, if successful, send you an encrypted email asking you to confirm the key import. **Check your email account and read the email that Launchpad sent you**. If your email client supports OpenPGP encryption, it will prompt you for the password you chose for the key when GPG generated it. Enter the password, then click the link to confirm that the key is yours.

If Launchpad encrypts the email, using your public key, so that it can be sure that the key is yours. If you are using 'Thunderbird', the default Ubuntu email client, you can install the 'Enigmail' plugin to easily decrypt the message. If your email software does not support OpenPGP encryption, copy the encrypted email's contents, type `gpg` in your terminal, then paste the email contents into your terminal window.

Once you open the link inside,

### (3.4) Confirm the GPG key in Launchpad

Launchpad will use the fingerprint to check the Ubuntu key server for your key and, if successful, send you an email asking you to confirm the key import. **Check your email account and read the email that Launchpad sent you**

If Launchpad encrypts the email, using your public key, so that it can be sure that the key is yours.
- If your email client supports OpenPGP encryption, it will prompt you for the password you chose for the key when GPG generated it. Enter the password, then click the link to confirm that the key is yours.
- If you are using 'Thunderbird', the default Ubuntu email client, you can install the 'Enigmail' plugin to easily decrypt the message.
- If your email software does not support OpenPGP encryption, copy the encrypted email's contents, type `gpg` in your terminal, then paste the email contents into your terminal window.

Otherwise Launchpad will ask to you for a signed text. To confirm that you own the OpenPGP key you want to use with Launchpad, you need to use the key to sign some text. Launchpad can then compare the signature on the text against the key you want to import. In this case follow the next steps:
1. Copy the confirmation text from the Launchpad page, paste it into a text editor and then save the file.
```
vi ./to-sign
```
2. Open a terminal (you may need to adjust the next few steps if you're not using Ubuntu or another GNU/Linux based system) and enter: `gpg --clearsign FILENAME` (replace FILENAME with the name of the text file you saved in step 1).
```
gpg --clearsign ./to-sign
```
3. Open the resultant text file (its name will be the same as your original file but with .asc appended) and paste its contents into the text-box on the Launchpad Confirm OpenPGP Key page. The signed text should look something like this:
```
cat ./to-sign.asc


-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

Please register 0837A887276EF500C9F3FC98AADB3BFE17758171 to the
Launchpad user apaz.  2022-05-24 05:45:05 UTC
-----BEGIN PGP SIGNATURE-----

iQIzBAEBCgAdFiEECDeohydu9QDJ8/yYqts7/hd1gXEFAmKMdDQACgkQqts7/hd1
gXGNPg/9FGnxt6VDzyJSqeVhm429D0cNCit0xtdVk1/t7HN8zDQf2V4ImKMldo9S
sIo7MQOhXdm5xGC4fp6WYuhAVNkRV/fhryjqtkFcRr42Rzq3NLQ9VDdypLkeh0x4
7brjNUOfnxKJTl5p+Z3b5zJoU7WQ+KcdxJoil5d77aGPd90dwvH0oseo8A+269Gz
jFxPsrnXmDrbKNk+2KEZc7MMgFXzZAXW4BRfOG5eMFLOe5URBDbzxkjrPemK8UrP
8xHUO4J0mRPsHsxDf0+K7eZLWqbHd50TYHNr42rllYpDDF7DXMSUardaHmI5G/tv
0iK+JGjkGduDrjKUBG2EkD/7j3EJlz2HEkdFG/UtIzomY2cEMXrNqZOW25xZ9cAm
jfLZjVRAoBrza8jwBtnVQxhDegpjD5NHFsJ1mrxng/7ypze4suqNCUAmXr2pv7VZ
TXjBhRNZE9ZxWreVBQ64HlGSfi/fSnMmGJptB+qOyjpl2gyYIfM5O5pa0A5NTu0Q
YtbrCyUNzs13LH5Hbjv+28IQD0IxjhnmhVAW0siVkrd3wv8jUVTN8EeOzYnbZIZX
fbSENme/M9Nk1fqsuG5OLkCOFPx8jCJHJWW4LO4PNFcu7m/fJKSUp938/tW/LAnb
6gcwiTgKkv+xwncVaXT17HnzeYq82Y1Ueb8aPPIy6h4qlnMgZZg=
=W59R
-----END PGP SIGNATURE-----
```

Finally click the 'Confirm' button and Launchpad will complete the import of your OpenPGP key.

### How to create a backup of your GPG keys

GPG keys, secret keys, subkeys and ownertrust can be exported and imported again using the following commands:

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

## (5) Create a SSH key and upload it to your Launchpad account

We will be using SSH to securely upload source packages to Launchpad.

SSH stands for Secure Shell, and it is a protocol that allows you to exchange data in a secure way over a network.

For more information see the [YourAccount/CreatingAnSSHKeyPair](https://help.launchpad.net/YourAccount/CreatingAnSSHKeyPair) page.

### (5.1) Create a SSH key

```sh
# ssh-keygen -t ed25519 -N "" -f "$HOME/.ssh/id_ed25519_YOURACCOUNT" -C "your_email@example.com"
ssh-keygen -t ed25519 -N "" -f "$HOME/.ssh/id_ed25519_AldoPaz_Launchpad" -C "aldo.paz@noemail.org"
chmod 400 "$HOME/.ssh/id_ed25519_AldoPaz_Launchpad"
```

Open the config file using `vi ~/.ssh/config` and configure it with the SSH key for Launchpad:

```sh
Host git.launchpad.net
    IdentityFile ~/.ssh/id_ed25519_AldoPaz_Launchpad
    # Here the account ID, the short name that appears after the '~' in the URL when you visit https://launchpad.net/~
    User apaz

```

Add the new key to the SSH agent:

```sh
ssh-add /home/apaz/.ssh/id_ed25519_AldoPaz_Launchpad
# Identity added: ...
```

### (5.2) Upload your SSH key to Launchpad

Add your SSH public key (the content of '$HOME/.ssh/id_ed25519_AldoPaz_Launchpad.pub' in my case) into [https://launchpad.net/~/+editsshkeys](https://launchpad.net/~/+editsshkeys).

## (6) Download the upstream tarball and test it

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

## (7) Debianize the source tree

**Many packages are packaged using only a text editor while imitating how other similar packages are packaged and consulting how the 'Debian policy' requires us to do**. This seems to me the most popular method for the real-life packaging activity.

Here I will be using 'dh-make'. It is a tool to convert a regular source code package into one formatted according to the requirements of the Debian Policy.

### (7.1) Install 'dh-make'

```sh
sudo apt-get install dh-make
```

### (7.2) Debianize from the tarball using the plugin

'dh_make' will generate a 'debian/' subdirectory and the necessary control files in the program source directory. Those control files are customized with the packagename and version extracted from the directory name. Also, as we will see below, it will create the '.orig.tar.gz' file making a copy of the tarball that we've already downloaded.

NOTE: 'dh_make' will obtain the username and e-mail address from 'DEBFULLNAME' and 'DEBEMAIL' respectively.

In the following command, we type 's' because we want to distribute a single binary, and 'y' to accept the values determined by the command.

```sh
# move to source tree
cd hello-2.12

# create
dh_make -f ../hello-2.12.tar.gz
# Type of package: (single, indep, library, python)
# [s/i/l/p]?
# Maintainer Name     : Aldo Paz
# Email-Address       : estebanpazutn@gmail.com
# Date                : Wed, 25 May 2022 14:57:49 -0300
# Package Name        : hello
# Version             : 2.12
# License             : blank
# Package Type        : single
# Are the details correct? [Y/n/q]
# Done. Please edit the files in the debian/ subdirectory now.

tree ..
# ..
# ├── hello-2.12
# ...
# │   └── debian
# │       ├── changelog
# │       ├── control
# │       ├── copyright
# │       ├── hello.cron.d.ex
# │       ├── hello.doc-base.EX
# │       ├── hello-docs.docs
# │       ├── manpage.1.ex
# │       ├── manpage.sgml.ex
# │       ├── manpage.xml.ex
# │       ├── postinst.ex
# │       ├── postrm.ex
# │       ├── preinst.ex
# │       ├── prerm.ex
# │       ├── README.Debian
# │       ├── README.source
# │       ├── rules
# │       ├── salsa-ci.yml.ex
# │       ├── source
# │       │   └── format
# │       └── watch.ex
# ├── hello_2.12.orig.tar.gz
# └── hello-2.12.tar.gz
```

The command above also copied 'hello-2.12.tar.gz' to 'hello_2.12.orig.tar.gz' because this is the file (a pristine copy of the upstream source package) required when creating a source package for debian.

### Requirement: the package-version/ directory name format

'dh_make' without parameters needs to be invoked inside a directory with the **package-version**/ name format. If the directory doesn't have this format, you will see a message like it:

```
For dh_make to find the package name and version, the current directory
needs to be in the format of <package>-<version>.  Alternatively use the
-p flag using the format <name>_<version> to override it.
The directory name you have specified is invalid!

Your current directory is:
/home/apaz/packaging-example/wrongname
Perhaps you could try going to directory where the sources are?

Please note that this change is necessary ONLY during the initial
Debianization with dh_make.  When building the package, dpkg-source
will gracefully handle almost any upstream tarball.
```

#### Required: the ../package-version.orig.tar.gz tarball is required

The original archive (.orig.tar.gz) is needed for other Debian tools to generate the diffs to the original sources required by the Debian packaging format. Unless there are reasons against it, this file should be the pristine upstream archive.

'dh_make' makes sure a original source archive (packagename_version.orig.tar.gz) exists in parent directory. The archive can either end with .gz or one of the other supported compression extensions such as bz2 or lzma. If no such file exists, the file specified with '-f' is copied in place.

If 'dh_make' doesn't find a way to get the '../packagename_version.orig.tar.gz' tarball, you will see a message like it:

```
Could not find hello_2.12.orig.tar.xz
Either specify an alternate file to use with -f,
or add --createorig to create one.
```

## (8) Customize the template files in debian/

At this point the maintainer should customize the template files generated inside 'debian/' by 'dh_make'. We could skip the customization here to show the possible errors that 'lintian', the static analysis tool for Debian packages, could throw when building a package. As I already documented how to fix some errors that 'lintian' could throw in another guide for creating packages for Debian specifically, we will proceed correctly here.

### (8.1) Remove unnecessary example files

Most of the files 'dh_make' adds are only needed for specialist packages (such as Emacs modules) so we can start by removing the optional example files:

```sh
cd ./debian
rm *ex *EX
cd ..
```

It deleted the following files:

```
hello.cron.d.ex
hello.doc-base.EX
manpage.1.ex
manpage.sgml.ex
manpage.xml.ex
postinst.ex
postrm.ex
preinst.ex
prerm.ex
salsa-ci.yml.ex
watch.ex
```

### (8.2) Customise each of the files

#### (8.2.1) Check and customize 'debian/source/format' if necessary

'debian/source/format' describes the version format of the source package and should be '3.0 (quilt)', the currenct standard format. Left it as is if this file has the correct value.

#### (8.2.2) Customize 'debian/control'

'debian/control' contains all the metadata of the package. The first paragraph describes the source package. The second and following paragraphs describe the binary packages to be built.

##### (8.2.2.1) Define the compat version in the 'Build-Depends:' section

NOTE: Much of the package building work is done by a series of scripts called 'debhelper'. The exact behaviour of debhelper changes with new major versions, the 'compat' number in the 'debian/control' file instructs 'debhelper' which version to act as. You will generally want to set this to the most recent version at the moment of packaging.

Example:

```
Build-Depends: debhelper-compat (= 13)
```

##### (8.2.2.2) Add other required build-dependencies in the 'Build-Depends:' section

'Build-Depends:' should have, at least, the debhelper compat dependency. We will probably need to add the packages needed to compile/build the application here too.

Example:

```
Build-Depends: debhelper-compat (= 13), autotools-dev
```

##### (8.2.2.3) Add other required run-dependencies in the 'Depends:' section

In the same way as 'Build-Depends:', the 'Depends:' section gives the dependencies to make your tool run.

Here we notice the variables '${shlibs:Depends}' and '${misc:Depends}'. Actually we can use them instead of adding manually a 'libxxx' (they are defined in the '.substvars' file).

#### (8.2.3) Customize 'debian/changelog' for Ubuntu

In 'debian/changelog' change the version number to an Ubuntu version.

For example, 2.10-0ubuntu1 means
- upstream version 2.10,
- Debian version 0,
- Ubuntu version 1.

Also change 'unstable' to the target development Ubuntu release (usually the current). I'll use 'hirsute' here.

NOTE: If we want to upload this source package to a PPA in Launchpad, then the target Ubuntu release in the changelog entry needs to specify one of the supported series. We have to check [https://launchpad.net/ubuntu/+ppas](https://launchpad.net/ubuntu/+ppas) to find the series currently supported by PPAs. If we specify a different series the build will fail.

##### How Ubuntu package version naming works

Basically each package will be in the form: **package-XubuntuY**. Where:

- **package** - is the name of the program/library.
- **X** - is the debian version of the package. if X=0 this means that there is no debian package (or that the ubuntu team has forked a debian package to a newer version than the one found in the debian repositories). ex: bzip2-1.0.3-0ubuntu2, in this example the debian package might be updated in the meantime and the ubuntu package will probably merged with it on the next version.
- **ubuntuY** - is the Yth ubuntu version of the debian package. If this is missing this mean that it is a clean, unchanged debian package. ex: gzip-1.3.5-12, is the original debian package included in ubuntu. If this is present it means that Ubuntu has taken the debian package and released it with some additional patches or bug fixes. ex: sudo-1.6.8p12-1ubuntu6, is the 6th version of the ubuntu package based on the debian version 1.6.8p12-1 of sudo.

Here some other examples:
- 2.6.0-1 - means that this is the 1st debian package of version 2.6.0. No ubuntu changes were included.
- 2.6.0-1ubuntu1 - means that this is the 1st ubuntu package based on the debian package version 2.6.0-1
- 2.6.0-0ubuntu1 - means that there was not a debian package yet and this is the 1st ubuntu version of package 2.6.0

#### (8.2.4) Customize 'debian/copyright'

'debian/copyright' needs to be filled in to follow the licence of the upstream source. Fixing a 'debian/copyright' file implies extracting all the different copyright licenses in the upstream's files.

'debmake' generated a copyright template including all the licenses extracted from the sources so we have to modify it to just include a simplified list with references to the corresponding complete license file in 'usr/share/common-licenses/'.

#### (8.2.5) Customize 'debian/rules'

'debian/rules' is the most complex file. This is a Makefile which compiles the code and turns it into a binary package. Fortunately most of the work is automatically done these days by 'debhelper' so the universal '%' target just runs the dh script which will run everything needed (probably).

#### (8.2.6) Customize or delete 'debian/README.Debian'

This file could be deleted if you think no notes regarding this package are needed.

#### (8.2.7) Customize or delete 'debian/README.source'

'debian/README.source' may include any information that would be helpful to someone modifying the source package. Maintainers are encouraged to document in a 'debian/README.source' file any source package with a particularly complex or unintuitive source layout or build system (for example, a package that builds the same source multiple times to generate different binary packages). See [4.14. Source package handling: debian/README.source in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-source.html#source-package-handling-debian-readme-source) for details.

#### (8.2.8) Customize or delete 'debian/*.docs'

Any 'debian/*.docs' file list documentation files to be installed into package. Supports substitution variables in compat 13 and later as documented in 'debhelper'. See `man dh_installdocs` for details.

#### (8.2.9) Example of changes

Changed 'debian/control' from:

```
Source: hello
Section: unknown
Priority: optional
Maintainer: Aldo Paz <estebanpazutn@gmail.com>
Build-Depends: debhelper-compat (= 13), autotools-dev
Standards-Version: 4.5.1
Homepage: <insert the upstream URL, if relevant>
#Vcs-Browser: https://salsa.debian.org/debian/hello
#Vcs-Git: https://salsa.debian.org/debian/hello.git
Rules-Requires-Root: no

Package: hello
Architecture: any
Depends: ${shlibs:Depends}, ${misc:Depends}
Description: <insert up to 60 chars description>
 <insert long description, indented with spaces>
```

to:

```
Source: hello
Section: devel
Priority: optional
Maintainer: Aldo Paz <estebanpazutn@gmail.com>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.5.1
Homepage: http://www.gnu.org/software/hello/
Rules-Requires-Root: no

Package: hello
Architecture: any
Depends: ${shlibs:Depends}, ${misc:Depends}
Description: example package based on GNU hello
 The GNU hello program produces a familiar, friendly greeting.  It
 allows non-programmers to use a classic computer science tool which
 would otherwise be unavailable to them.
 .
 Seriously, though: this is an example of how to do a Debian package.
 It is the Debian version of the GNU Project's `hello world' program
 (which is itself an example for the GNU Project).
```

NOTE: Since compatibility level 10, 'debhelper' enables the autoreconf sequence by default. It is therefore not necessary to specify build-dependencies on 'dh-autoreconf' or 'autotools-dev' and they can be removed.

NOTE: The extended description (the lines after the first line of the "Description:" section) is required and must not be empty. See [extended-description-is-empty](https://lintian.debian.org/tags/extended-description-is-empty) and [3.4. The description of a package in the Debian Policy Manual](https://www.debian.org/doc/debian-policy/ch-binary.html#the-description-of-a-package) for details.

Changed 'debian/changelog' from:

```
hello (2.12-1) unstable; urgency=medium

  * Initial release (Closes: #nnnn)  <nnnn is the bug number of your ITP>

 -- Aldo Paz <estebanpazutn@gmail.com>  Thu, 26 May 2022 20:08:29 -0300
```

to:

```
hello (2.12-0ubuntu1) hirsute; urgency=low

  * Initial packaging.

 -- Aldo Paz <estebanpazutn@gmail.com>  Thu, 26 May 2022 20:08:29 -0300
```

Changed 'debian/rules' from:

```makefile
#!/usr/bin/make -f
# See debhelper(7) (uncomment to enable)
# output every command that modifies files on the build system.
#export DH_VERBOSE = 1


# see FEATURE AREAS in dpkg-buildflags(1)
#export DEB_BUILD_MAINT_OPTIONS = hardening=+all

# see ENVIRONMENT in dpkg-buildflags(1)
# package maintainers to append CFLAGS
#export DEB_CFLAGS_MAINT_APPEND  = -Wall -pedantic
# package maintainers to append LDFLAGS
#export DEB_LDFLAGS_MAINT_APPEND = -Wl,--as-needed


%:
	dh $@


# dh_make generated override targets
# This is example for Cmake (See https://bugs.debian.org/641051 )
#override_dh_auto_configure:
#	dh_auto_configure -- \
#	-DCMAKE_LIBRARY_PATH=$(DEB_HOST_MULTIARCH)
```

to:

```makefile
#!/usr/bin/make -f

%:
	dh $@

override_dh_auto_clean:
	[ ! -f Makefile ] || dh_auto_clean

# skip processing configure.ac because missing build-aux/git-version-gen.
override_dh_autoreconf:
```

Changed 'debian/README.Debian', I deleted it.

Changed 'debian/README.source', I deleted it.

Changed 'debian/copyright' to:

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

## (9) Build the package

'builddeb' is the command to build the package. The '-us' and '-uc' options tell it there is no need to GPG sign the package. The result will be placed in the parent directory.

```sh
# Build
debuild -us -uc
#...

# List the content of the binary package
lesspipe ../hello_2.12-0ubuntu1_amd64.deb
#...
```

## (10) Install the package and check it works

We can use `dpkg --install <deb-file>` or `sudo debi -a amd64`.

In the case of 'debi', it will look for the .changes file corresponding to the current package version (by determining the name and version number from the changelog, and the architecture in the same way as 'dpkg-buildpackage' does). It then runs 'debpkg -i' on every '.deb' archive listed in the '.changes' file to install them, assuming that all of the '.deb' archives live in the same directory as the '.changes' file. Note that you probably don't want to run this program on a '.changes' file relating to a different architecture after cross-compiling the package!. See `man debi` for details.

```sh
# Install
sudo dpkg --install ../hello_2.10-0ubuntu1_amd64.deb
# or
# sudo debi -a amd64

# Test it
# hello
# hello -g Aldo
# man hello

# Uninstall
sudo apt-get remove hello
```

## (11) Fix errors with 'lintian'

### (11.1) Run 'lintian'

Even if it builds the '.deb' binary package, your packaging may have bugs. Many errors can be automatically detected by our tool 'lintian' which can be run on
- the source '.dsc' metadata file,
- '.deb' binary packages
- or '.changes' file:

```sh
# check .dsc
lintian ../hello_2.12-0ubuntu1.dsc

# check .deb
lintian ../hello_2.12-0ubuntu1_amd64.deb

# check .changes
lintian ../hello_2.12-0ubuntu1_amd64.changes
```

NOTE: To see verbose description of the problems we can use the '--info' flag or the 'lintian-info' command. For Python packages, there is also a 'lintian4python' tool that provides some additional lintian checks.

### (11.2) Rebuild the package

After making a fix to the packaging you can rebuild using '-nc' (no clean) without having to build from scratch:

```sh
# Build
debuild -us -uc -nc
#...
```

## (12) Build a signed, source-only .changes file

Since we are going to upload to a PPA (Personal Package Archive) the upload will need to be signed to allow Launchpad to verify that the upload comes from you.

NOTE: For signing to work you need to have set up GPG (step 3).

'debuild' (from 'devscripts') will invoke 'debsign' after completing a successful build (and when not invoked with the '-uc' and '-us' options). This is easier than running 'debsign' yourself, and usually works well, as long as you don't sign the packages on a different machine from the one used to build them.

NOTE: the upload file (_source.changes) will be signed because the '-us' and '-uc' flags are not passed to 'debuild' like they were before.

Also note **we can upload signed, source packages only to Launchpad**. It means we have to make a signed, source-only '_source.changes' file (aka. the upload file) that Launchpad will accept.

NOTE: At least specifically said, 'debuild' will generate both binary and source packages and so the '.changes' file it generates will contain both source and binary and Launchpad will reject it. To generate a source-only '_source.changes' file we have to add the '-S' flag. Also you would like to add '-d' to not check build dependencies and conflicts, because we already know the package builds correctly locally.

```sh
# Build source-only package and sign it
debuild -S -d
# ...
# Now signing changes and any dsc files...
#  signfile dsc hello_2.12-0ubuntu1.dsc Aldo Paz <estebanpazutn@gmail.com>

#  fixup_buildinfo hello_2.12-0ubuntu1.dsc hello_2.12-0ubuntu1_source.buildinfo
#  signfile buildinfo hello_2.12-0ubuntu1_source.buildinfo Aldo Paz <estebanpazutn@gmail.com>

#  fixup_changes dsc hello_2.12-0ubuntu1.dsc hello_2.12-0ubuntu1_source.changes
#  fixup_changes buildinfo hello_2.12-0ubuntu1_source.buildinfo hello_2.12-0ubuntu1_source.changes
#  signfile changes hello_2.12-0ubuntu1_source.changes Aldo Paz <estebanpazutn@gmail.com>

# Successfully signed dsc, buildinfo, changes files
```

This command also generates three files:

```sh
ls ..
# ...
# hello_2.12-0ubuntu1_source.changes
# hello_2.12-0ubuntu1_source.build
# hello_2.12-0ubuntu1_source.buildinfo
```

See [Uploading in the ubuntu wiki](https://wiki.ubuntu.com/UbuntuDevelopment/Uploading) for more information.

### Alternative way of signing using 'debsign' manually

TODO

### How 'debsign' signs the source package with GPG

The signature consists of two parts. The first is the signature on the '.dsc' file, which is the signature of the source package itself. The second is the signature on the '_source.changes' file, which is the signature of the upload.

**'debsign' will take the 'Changed-By' entry from the '_source.changes' file, and ask GPG to sign the two parts with the key that has a UID matching that value. The UID must be byte-for-byte identical to the 'Changed-By' value (including the key "comment"), so any changes in the name or email address (even down to whitespace) will cause the match to fail. If that happens then GPG will report that it can't find the needed secret key**.

## (13) Upload the source package to a PPA

Once we've created our source package, we could upload it and Launchpad will build binaries and then host them in our own apt repository (it is a PPA, a a Personal Package Archive). It also gives an easy way for us and others to test the binary packages.

We will need to set up a PPA in Launchpad and then upload the source package with 'dput'.

See [Packaging/PPA](https://help.launchpad.net/Packaging/PPA) for details.

### (13.1) Create/Activate a PPA in Launchpad

NOTE: Every individual and team in Launchpad can have one or more PPAs, each with its own URL.

NOTE: Launchpad generates a unique key for each PPA and uses it to sign any packages built in that PPA. This means that people downloading/installing packages from a PPA can verify their source. After activating a PPA, uploading its first package causes Launchpad to start generating the key, which can take up to a couple of hours to complete.

In my case I have to go to [https://launchpad.net/~apaz/+activate-ppa](https://launchpad.net/~apaz/+activate-ppa) to create a new PPA.

```sh
# bzr push lp:~<lp-username>/+junk/hello-package
```

TODO

### (13.2) Upload the source package to the PPA

NOTE: Launchpad does not allow uploading pre-built binary packages.

NOTE: Each PPA gets 2 GiB of disk space. If more space is needed for a particular PPA, ask for it on [https://answers.launchpad.net/soyuz](https://answers.launchpad.net/soyuz).

Packages we publish in our PPA will remain there until
- we remove them,
- they're superseded by another package that we upload
- or the version of Ubuntu against which they're built becomes obsolete.

```sh
# dput ppa:<lp-username>/<ppa-name> hello_2.10-0ubuntu1.changes
```

TODO

See [Packaging/PPA/Uploading](https://help.launchpad.net/Packaging/PPA/Uploading) for details.

### (13.3) Test installing the package from the PPA

Ubuntu users can install your packages in just the same way they install standard Ubuntu packages and they'll automatically receive updates as and when we make them.

TODO

See [Packaging/PPA/InstallingSoftware](https://help.launchpad.net/Packaging/PPA/InstallingSoftware) for details.

### The PPAs' main site

See [https://launchpad.net/ubuntu/+ppas](https://launchpad.net/ubuntu/+ppas) to find:
- The most active PPAs,
- and the series currently supported.

## (13) Publish the code at Launchpad

Using Launchpad, we can publish Bazaar branches or Git repositories of your code and, optionally, associate them with [Launchpad's projects](https://help.launchpad.net/Projects/Registering). You can also mirror Bazaar branches that are hosted elsewhere on the internet and even import git, Subversion and CVS repositories into Bazaar branches.

**We do not need to register a Launchpad's project to publish into a PPA**, so here we will proceed by uploading our changes into a "personal" repository with no particular connection to any project or package (like "+junk" in Launchpad's Bazaar code hosting).

In the following steps we will upload our code in a personal git repository. We can see our git repositories at https://code.launchpad.net/~OWNER/+git, for example [https://code.launchpad.net/~apaz/+git](https://code.launchpad.net/~apaz/+git).

See [Code/Git in Launchpad](https://help.launchpad.net/Code/Git) for details about managing git repositories on Launchpad.

### (13.1) Initialize a git repository at a clean project

First make sure you have a clean source package with the debian directory:

```sh
cd ..
mv hello-2.12 temp
tar xzf hello-2.12.tar.gz
cd hello-2.12
mv ../temp/debian .
git init
git add .
git commit -m "Initial commit"
```

### (13.2) Push into a personal respository on Launchpad

```sh
# git remote add launchpad git+ssh://USER@git.launchpad.net/~OWNER/+git/REPOSITORY
git remote add launchpad git+ssh://apaz@git.launchpad.net/~apaz/+git/hello
git push launchpad master
# Enumerating objects: 469, done.
# Counting objects: 100% (469/469), done.
# Delta compression using up to 4 threads
# Compressing objects: 100% (465/465), done.
# Writing objects: 100% (469/469), 1.09 MiB | 60.00 KiB/s, done.
# Total 469 (delta 125), reused 0 (delta 0), pack-reused 0
# remote: Resolving deltas: 100% (125/125), done.
# To git+ssh://git.launchpad.net/~apaz/+git/hello
#  * [new branch]      master -> master
```

With the 'push' command above, the new repository should be accessible on 'https://code.launchpad.net/~apaz/+git/hello' and the corresponding cgit page on 'https://git.launchpad.net/~apaz/+git/hello'.

#### Defining an abbreviation for repositories hosted on Launchpad

Edit '~/.gitconfig' and add these lines, where USER is your Launchpad username:

```conf
[url "git+ssh://USER@git.launchpad.net/"]
        insteadof = lp:
```

Example:

```conf
[url "git+ssh://apaz@git.launchpad.net/"]
        insteadof = lp:
```

For this example, this allows to type:

```sh
git remote add launchpad lp:~apaz/+git/hello
```

instead of:

```sh
git remote add launchpad git+ssh://apaz@git.launchpad.net/~apaz/+git/hello
```

#### Repository URLs

Every Git repository hosted on Launchpad has a full "canonical" URL of one of these forms (these are the versions you'd use in a web browser; you only need to change the scheme and host parts for the command-line Git client):

- https://code.launchpad.net/~OWNER/PROJECT/+git/REPOSITORY - This identifies a REPOSITORY for an upstream PROJECT.
- https://code.launchpad.net/~OWNER/DISTRIBUTION/+source/SOURCE/+git/REPOSITORY - This identifies a REPOSITORY for a SOURCE package in a DISTRIBUTION.
- https://code.launchpad.net/~OWNER/+git/REPOSITORY - This identifies a "personal" REPOSITORY with no particular connection to any project or package (like "+junk" in Launchpad's Bazaar code hosting).

These are unique, but can involve quite a lot of typing, and in most cases there's no need for more than one repository per owner and target (project or package). Launchpad therefore has the notion of "default repositories". A repository can be the default for a target, in which case it has one of these forms:

- https://code.launchpad.net/PROJECT - This is the default repository for an upstream project.
- https://code.launchpad.net/DISTRIBUTION/+source/SOURCE - This is the default repository for a source package in a distribution.

Or a repository can be a person's or a team's default for a target, in which case it has one of these forms:

- https://code.launchpad.net/~OWNER/PROJECT - This is an owner's default repository for an upstream project.
- https://code.launchpad.net/~OWNER/DISTRIBUTION/+source/SOURCE - This is an owner's default repository for a source package in a distribution.

It is expected that projects hosting their code on Launchpad will normally have their primary repository set as the default for the project, and contributors will normally push to branches in owner-default repositories. The extra flexibility with named repositories allows for situations such as separate private repositories containing embargoed security fixes.

## (14) Create a recipe in Launchpad

A "recipe" in Launchpad is a description of the steps needed to construct a package from a set of Bazaar or Git branches. It allows to automatize the packaging and then host the builded packages.

The format of a recipe specifies:

- Which branch to use for the _source code_: trunk branch, beta branch, etc.
- Which branch to use for the _packaging information_: separate branch, Ubuntu branch, etc.
- The correct _package version_ (so people can still upgrade to the new stable version when it's released).
- What to modify to make the source build properly.

See [Packaging/SourceBuilds/GettingStarted](https://help.launchpad.net/Packaging/SourceBuilds/GettingStarted) and [Packaging/SourceBuilds/Recipes](https://help.launchpad.net/Packaging/SourceBuilds/Recipes) for details.

TODO

## How to set up 'pbuilder'

'pbuilder' allows you to build packages locally on your machine. It serves a couple of purposes:
- The build will be done in a minimal and clean environment. This helps you make sure your builds succeed in a reproducible way, but without modifying your local system.
- There is no need to install all necessary build dependencies locally.
- You can set up multiple instances for various Ubuntu and Debian releases.

```sh
# pbuilder-dist <release> create
pbuilder-dist hirsute create
```

where \<release\> is for example xenial, zesty, artful or in the case of Debian maybe sid or buster.

This will take a while as it will download all the necessary packages for a "minimal installation". These will be cached though.

### Building a binary package using 'pbuilder'

Having checked that the package builds locally (step 11) you should ensure it builds on a clean system using 'pbuilder'.

```sh
cd ../build-area

# pbuilder-dist <release> build <dsc-file>
pbuilder-dist trusty build ../hello_2.12-0ubuntu1.dsc
```

## Using a 'LXD container' to packaging in an latest development version of Ubuntu

TODO

## Reference

- [NewPackages in the Ubuntu wiki](https://wiki.ubuntu.com/UbuntuDevelopment/NewPackages)
- [Packaging/PPA in Launchpad](https://help.launchpad.net/Packaging/PPA)
