- [Creating packages for Ubuntu](#creating-packages-for-ubuntu)
  - [(1) Create and configure your account on Launchpad](#1-create-and-configure-your-account-on-launchpad)
    - [(1.1) Get a Launchpad account and account ID](#11-get-a-launchpad-account-and-account-id)
  - [(2) Install packaging-related software](#2-install-packaging-related-software)
    - [(2.1) Install the essential packages](#21-install-the-essential-packages)
    - [(2.2) Export configuration variables for these tools](#22-export-configuration-variables-for-these-tools)
  - [(3) Create a GPG key and upload it to your Launchpad account](#3-create-a-gpg-key-and-upload-it-to-your-launchpad-account)
    - [(3.1) Create a GPG key](#31-create-a-gpg-key)
    - [(3.2) Upload the public part of your GPG key to the Ubuntu keyserver](#32-upload-the-public-part-of-your-gpg-key-to-the-ubuntu-keyserver)
    - [(3.3) Upload your GPG key to Launchpad](#33-upload-your-gpg-key-to-launchpad)
    - [(3.4) Confirm the GPG key in Launchpad](#34-confirm-the-gpg-key-in-launchpad)
    - [How to create a backup of your GPG keys](#how-to-create-a-backup-of-your-gpg-keys)
      - [Copying the ~/.gnupg](#copying-the-gnupg)
      - [Export and import commands](#export-and-import-commands)
  - [(5) Create a SSH key and upload it to your Launchpad account](#5-create-a-ssh-key-and-upload-it-to-your-launchpad-account)
    - [(5.1) Create a SSH key](#51-create-a-ssh-key)
    - [(5.2) Upload your SSH key to Launchpad](#52-upload-your-ssh-key-to-launchpad)
  - [(6) Download the upstream tarball and test it](#6-download-the-upstream-tarball-and-test-it)
  - [(7) Debianize the source tree](#7-debianize-the-source-tree)
    - [(7.1) Install 'dh-make'](#71-install-dh-make)
    - [(7.2) Debianize from the tarball using the plugin](#72-debianize-from-the-tarball-using-the-plugin)
    - [Requirement: the package-version/ directory name format](#requirement-the-package-version-directory-name-format)
      - [Required: the ../package-version.orig.tar.gz tarball is required](#required-the-package-versionorigtargz-tarball-is-required)
  - [(8) Customize the template files in debian/](#8-customize-the-template-files-in-debian)
    - [(8.1) Remove unnecessary example files](#81-remove-unnecessary-example-files)
    - [(8.2) Customise each of the files](#82-customise-each-of-the-files)
      - [(8.2.1) Check and customize 'debian/source/format' if necessary](#821-check-and-customize-debiansourceformat-if-necessary)
      - [(8.2.2) Customize 'debian/control'](#822-customize-debiancontrol)
        - [(8.2.2.1) Define the compat version in the 'Build-Depends:' section](#8221-define-the-compat-version-in-the-build-depends-section)
        - [(8.2.2.2) Add other required build-dependencies in the 'Build-Depends:' section](#8222-add-other-required-build-dependencies-in-the-build-depends-section)
        - [(8.2.2.3) Add other required run-dependencies in the 'Depends:' section](#8223-add-other-required-run-dependencies-in-the-depends-section)
      - [(8.2.3) Customize 'debian/changelog' for Ubuntu](#823-customize-debianchangelog-for-ubuntu)
        - [How Ubuntu package version naming works](#how-ubuntu-package-version-naming-works)
      - [(8.2.4) Customize 'debian/copyright'](#824-customize-debiancopyright)
      - [(8.2.5) Customize 'debian/rules'](#825-customize-debianrules)
      - [(8.2.6) Customize or delete 'debian/README.Debian'](#826-customize-or-delete-debianreadmedebian)
      - [(8.2.7) Customize or delete 'debian/README.source'](#827-customize-or-delete-debianreadmesource)
      - [(8.2.8) Customize or delete 'debian/*.docs'](#828-customize-or-delete-debiandocs)
      - [(8.2.9) Example of changes](#829-example-of-changes)
  - [(9) Build the package](#9-build-the-package)
  - [(10) Install the package and check it works](#10-install-the-package-and-check-it-works)
  - [(11) Fix errors with 'lintian'](#11-fix-errors-with-lintian)
    - [(11.1) Run 'lintian'](#111-run-lintian)
    - [(11.2) Rebuild the package](#112-rebuild-the-package)
  - [(12) Build a signed, source-only .changes file](#12-build-a-signed-source-only-changes-file)
    - [Options when creating a "brand new package" vs "derivative of a package that's already in Ubuntu's primary archive"](#options-when-creating-a-brand-new-package-vs-derivative-of-a-package-thats-already-in-ubuntus-primary-archive)
    - [Alternative way of signing using 'debsign' manually](#alternative-way-of-signing-using-debsign-manually)
    - [How 'debsign' signs the source package with GPG](#how-debsign-signs-the-source-package-with-gpg)
  - [(13) Publishing into a PPA on Launchpad - Alternative 1 - Upload the source package to a PPA](#13-publishing-into-a-ppa-on-launchpad---alternative-1---upload-the-source-package-to-a-ppa)
    - [(13.1) Create/Activate a PPA in Launchpad](#131-createactivate-a-ppa-in-launchpad)
    - [The PPAs' main site](#the-ppas-main-site)
    - [(13.2) Upload the source package to the PPA](#132-upload-the-source-package-to-the-ppa)
    - [(13.3) Test installing the package from the PPA](#133-test-installing-the-package-from-the-ppa)
    - [How to remove a PPA repository](#how-to-remove-a-ppa-repository)
  - [(14) Publishing into a PPA on Launchpad - Alternative 2 - Create the source package using a recipe (aka. daily build)](#14-publishing-into-a-ppa-on-launchpad---alternative-2---create-the-source-package-using-a-recipe-aka-daily-build)
    - [(14.1) Publish the code on Launchpad](#141-publish-the-code-on-launchpad)
      - [(14.1.1) Maintain a Debian package in git with 'git-buildpackage' (gbp)](#1411-maintain-a-debian-package-in-git-with-git-buildpackage-gbp)
        - [Adding entries to 'debian/changelog' using 'gbp'](#adding-entries-to-debianchangelog-using-gbp)
        - [Git clone, commit, and tag using 'gbp'](#git-clone-commit-and-tag-using-gbp)
      - [(14.1.3) Test building the package with 'gbp'](#1413-test-building-the-package-with-gbp)
      - [(14.1.2) Push into a personal respository on Launchpad](#1412-push-into-a-personal-respository-on-launchpad)
        - [Defining an abbreviation for repositories hosted on Launchpad](#defining-an-abbreviation-for-repositories-hosted-on-launchpad)
        - [Repository URLs](#repository-urls)
    - [(14.2) Write the recipe](#142-write-the-recipe)
      - [Example recipe for merging two different repositories/branches](#example-recipe-for-merging-two-different-repositoriesbranches)
    - [(14.3) Install 'git-build-recipe' and test the recipe](#143-install-git-build-recipe-and-test-the-recipe)
    - [(14.4) Create the recipe on Launchpad](#144-create-the-recipe-on-launchpad)
  - [Packaging using a Debian uscan file ('debian/watch')](#packaging-using-a-debian-uscan-file-debianwatch)
    - [Testing and troubleshooting with 'uscan'](#testing-and-troubleshooting-with-uscan)
    - [Add a new upstream release to a repository using 'gbp' and 'uscan'](#add-a-new-upstream-release-to-a-repository-using-gbp-and-uscan)
  - [Layouts for Git packaging repositories](#layouts-for-git-packaging-repositories)
    - [DEP-14 style or prestine-tar layout](#dep-14-style-or-prestine-tar-layout)
    - [The overlay layout](#the-overlay-layout)
  - [Configuring 'git-buildpackage' (gbp)](#configuring-git-buildpackage-gbp)
    - [Common configuration for prestine-tar layout](#common-configuration-for-prestine-tar-layout)
    - [Common configuration for overlay layout](#common-configuration-for-overlay-layout)
  - [Renaming the orig tarball properly using 'mk-origtargz'](#renaming-the-orig-tarball-properly-using-mk-origtargz)
  - [Downloading the orig tarball of a Debian package from various sources using 'origtargz'](#downloading-the-orig-tarball-of-a-debian-package-from-various-sources-using-origtargz)
  - [About Launchpad's builders](#about-launchpads-builders)
  - [How to set up 'pbuilder'](#how-to-set-up-pbuilder)
    - [Building a binary package using 'pbuilder'](#building-a-binary-package-using-pbuilder)
  - [Using a 'LXD container' to packaging in an latest development version of Ubuntu](#using-a-lxd-container-to-packaging-in-an-latest-development-version-of-ubuntu)
  - [Reference](#reference)

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
sudo apt install gnupg ubuntu-dev-tools
```

It installs:
- 'gnupg' - GNU Privacy Guard contains tools you will need to create a cryptographic key with which you will sign files you want to upload to Launchpad.
- 'ubuntu-dev-tools' (and 'devscripts', a direct dependency) - a collection of tools that make many packaging tasks easier.

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

Find more information at [YourAccount/ImportingYourPGPKey](https://help.launchpad.net/YourAccount/ImportingYourPGPKey).

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

#### Copying the ~/.gnupg

The easiest way would be to grab the entire GnuPG directory '~/.gnupg', it contains all private keys you have, as well as the public keyring and other useful data (trustdb, etc.):

```sh
cp -r ~/.gnupg <dst-location>
```

#### Export and import commands

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

'dh_make' will generate a 'debian/' subdirectory and the necessary control files in the program source directory. Those control files are customized with the packagename and version extracted from the directory name. Also, as we will see below, it will create the original source file ('.orig.tar.gz') by making a copy of the upstream tarball that we've already downloaded.

NOTE: 'dh_make' will obtain the username and e-mail address from 'DEBFULLNAME' and 'DEBEMAIL' respectively.

In the following command, we type 's' because we want to distribute a single binary, and 'y' to accept the values determined by the command.

```sh
# move to source tree
cd hello-2.12

# create
dh_make -f ../hello-2.12.tar.gz
# Type of package: (single, indep, library, python)
# [s/i/l/p]?    <--- ANSWER: s
# Maintainer Name     : Aldo Paz
# Email-Address       : aldo.paz@noemail.org
# Date                : Wed, 25 May 2022 14:57:49 -0300
# Package Name        : hello
# Version             : 2.12
# License             : blank
# Package Type        : single
# Are the details correct? [Y/n/q]    <--- ANSWER: y
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

The command above also copied 'hello-2.12.tar.gz' to 'hello_2.12.orig.tar.gz'. This '.orig.tar.gz' file is known as the original source (a pristine copy of the upstream source package) and is the file required when creating a source package for debian.

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

'dh_make' makes sure a original source archive (packagename_version.orig.tar.gz) exists in parent directory. The archive can either end with .gz or one of the other supported compression extensions such as bz2 or lzma. **If no such file exists, the file specified with '-f' is copied in place**.

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
rm ./debian/*ex ./debian/*EX
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

Also change 'unstable' to the target development Ubuntu release (usually the current). I'll use 'jammy' here.

Finally change the urgency to 'urgency=medium', 'low' isn't really used in Ubuntu.

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
Maintainer: Aldo Paz <aldo.paz@noemail.org>
Build-Depends: debhelper-compat (= 13), autotools-dev
Standards-Version: 4.6.0
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
Maintainer: Aldo Paz <aldo.paz@noemail.org>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.6.0
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

 -- Aldo Paz <aldo.paz@noemail.org>  Wed, 08 Jun 2022 22:42:27 -0300
```

to:

```
hello (2.12-0ubuntu1) jammy; urgency=low

  * Initial packaging.

 -- Aldo Paz <aldo.paz@noemail.org>  Wed, 08 Jun 2022 22:42:27 -0300
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

I deleted ex and EX files from debian/ directory with `rm ./debian/*ex ./debian/*EX`.

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

If we need to run the clean target first, we can run `debuild -- clean` (internally it invokes `dpkg-buildpackage --rules-target clean -us -uc -ui` which calls `debian/rules clean`).

## (12) Build a signed, source-only .changes file

**We can upload signed, source packages only to Launchpad**. Launchpad builds the packages onsite, and does not accept 'deb' files.

Since we are going to upload to a PPA (Personal Package Archive), the upload needs to be signed to allow Launchpad to verify that the upload comes from you. It means we have to make a signed, source-only '_source.changes' file (aka. the upload file) that Launchpad will accept.

NOTE: For signing to work we need to have set up GPG (step 3).

'debuild' (from 'devscripts') will invoke 'debsign' after completing a successful build (and when not invoked with the '-uc' and '-us' options). This is easier than running 'debsign' yourself, and usually works well, as long as you don't sign the packages on a different machine from the one used to build them.

The correct command for creating the Debian package source is `debuild -S`. At least specifically said, 'debuild' will generate both binary and source packages and so the '.changes' file it generates will contain both source and binary and Launchpad will reject it. To generate a source-only '_source.changes' file we have to add the '-S' flag (and the '-sa' option just to make sure the '.orig.tar.gz' file is listed in the '_source.changes' file too). Also we would like to add '-d' to not check build dependencies and conflicts, because we already know the package builds correctly locally.

NOTE: the upload file (_source.changes) will be signed because the '-us' and '-uc' flags are not passed to 'debuild' like they were before.

```sh
# Build source-only package and sign it
debuild -S -sa -d
# ...
# Now signing changes and any dsc files...
#  signfile dsc hello_2.12-0ubuntu1.dsc Aldo Paz <aldo.paz@noemail.org>

#  fixup_buildinfo hello_2.12-0ubuntu1.dsc hello_2.12-0ubuntu1_source.buildinfo
#  signfile buildinfo hello_2.12-0ubuntu1_source.buildinfo Aldo Paz <aldo.paz@noemail.org>

#  fixup_changes dsc hello_2.12-0ubuntu1.dsc hello_2.12-0ubuntu1_source.changes
#  fixup_changes buildinfo hello_2.12-0ubuntu1_source.buildinfo hello_2.12-0ubuntu1_source.changes
#  signfile changes hello_2.12-0ubuntu1_source.changes Aldo Paz <aldo.paz@noemail.org>

# Successfully signed dsc, buildinfo, changes files
```

The options means:
 - '-S' - Equivalent to '--build=source'.
 - '-sa' - Forces the inclusion of the original source (the '.orig.tar.gz' file).
 - '-d' - Do not check build dependencies and conflicts.

This command also generates three files, specific for the source-only package build:

```sh
ls ..
# ...
# hello_2.12-0ubuntu1_source.changes
# hello_2.12-0ubuntu1_source.build
# hello_2.12-0ubuntu1_source.buildinfo
```

See [Packaging/PPA/BuildingASourcePackage](https://help.launchpad.net/Packaging/PPA/BuildingASourcePackage) and [Packaging/PPA/Uploading](https://wiki.ubuntu.com/UbuntuDevelopment/Uploading) for more information.

### Options when creating a "brand new package" vs "derivative of a package that's already in Ubuntu's primary archive"

How we build your package depends on whether we're creating a brand new package or we're creating a derivative of a package that's already in Ubuntu's primary archive.

If we're creating an alternative version of a package that's already in Ubuntu's primary archive, we don't need to upload the '.orig.tar.gz' file (the original source).

So, the options we can use to build a source-only package for ubuntu are:
- `debuild -S -sa` - brand new package with no existing version in Ubuntu's repositories (will be uploaded with the '.orig.tar.gz' file).
- `debuild -S -sd` - builds an alternative version of an existing package (will be uploaded without the .orig.tar.gz file).

NOTE: If we get the error "clearsign failed: secret key not available" when signing the changes file, use an additional option '-k[key_id]' when calling 'debuild'. Use `gpg --list-secret-keys` to get the key ID. Look for a line like "sec 12345/12ABCDEF"; the part after the slash is the key ID.

### Alternative way of signing using 'debsign' manually

TODO

### How 'debsign' signs the source package with GPG

The signature consists of two parts. The first is the signature on the '.dsc' file, which is the signature of the source package itself. The second is the signature on the '_source.changes' file, which is the signature of the upload.

**'debsign' will take the 'Changed-By' entry from the '_source.changes' file, and ask GPG to sign the two parts with the key that has a UID matching that value. The UID must be byte-for-byte identical to the 'Changed-By' value (including the key "comment"), so any changes in the name or email address (even down to whitespace) will cause the match to fail. If that happens then GPG will report that it can't find the needed secret key**.

## (13) Publishing into a PPA on Launchpad - Alternative 1 - Upload the source package to a PPA

NOTE: Before to proceed, we should have confirmed that our build works by testing it locally.

Once we've created our source package, we could upload it and Launchpad will build binaries and then host them in our own apt repository (it is a PPA, a a Personal Package Archive). It also gives an easy way for us and others to test the binary packages.

We will need to set up a PPA in Launchpad and then upload the source package with 'dput'.

See [Packaging/PPA](https://help.launchpad.net/Packaging/PPA) for details.

### (13.1) Create/Activate a PPA in Launchpad

Every individual and team in Launchpad can have one or more PPAs, each with its own URL. Before we can start using a PPA, whether it's your own or it belongs to a team, we need to activate it on our profile page or the team's overview page. In my case I have to go to [https://launchpad.net/~apaz/+activate-ppa](https://launchpad.net/~apaz/+activate-ppa) (or by clicking "Create a new PPA" link inside our profile page).

NOTE: If we already have one or more PPAs, this is also where we'll be able to create additional archives.

NOTE: Launchpad generates a unique key for each PPA and uses it to sign any packages built in that PPA. This means that people downloading/installing packages from a PPA can verify their source. **After activating a PPA, uploading its first package causes Launchpad to start generating the key, which can take up to a couple of hours to complete**.

For this example I've created a PPA called 'test' located on [https://launchpad.net/~apaz/+archive/ubuntu/test](https://launchpad.net/~apaz/+archive/ubuntu/test).

### The PPAs' main site

See [https://launchpad.net/ubuntu/+ppas](https://launchpad.net/ubuntu/+ppas) to find:
- The most active PPAs,
- and the series currently supported.

### (13.2) Upload the source package to the PPA

Once activated, we can upload our source packages with a command like `dput ppa:<lp-username>/<ppa-name> <source.changes>`. For this example the format will be `dput ppa:apaz/test <source.changes>`.

NOTE: Launchpad does not allow uploading pre-built binary packages.

NOTE: Each PPA gets 2 GiB of disk space. If more space is needed for a particular PPA, ask for it on [https://answers.launchpad.net/soyuz](https://answers.launchpad.net/soyuz).

Packages we publish in our PPA will remain there until
- we remove them,
- they're superseded by another package that we upload
- or the version of Ubuntu against which they're built becomes obsolete.

```sh
cd ..

dput ppa:apaz/test hello_2.12-0ubuntu1_source.changes
# D: Splitting host argument out of  ppa:apaz/test.
# D: Setting host argument.
# Checking signature on .changes
# gpg: /home/apaz/repos/temp/hello_2.12-0ubuntu1_source.changes: Valid signature from AADB3BFE17758171
# Checking signature on .dsc
# gpg: /home/apaz/repos/temp/hello_2.12-0ubuntu1.dsc: Valid signature from AADB3BFE17758171
# Uploading to ppa (via ftp to ppa.launchpad.net):
#   Uploading hello_2.12-0ubuntu1.dsc: done.
#   Uploading hello_2.12.orig.tar.gz: done.
#   Uploading hello_2.12-0ubuntu1.debian.tar.xz: done.
#   Uploading hello_2.12-0ubuntu1_source.buildinfo: done.
#   Uploading hello_2.12-0ubuntu1_source.changes: done.
# Successfully uploaded packages.
```

After this, If everything goes well, we should receive an email with a subject similar to this: "\[~apaz/ubuntu/test/jammy\] hello 2.12-0ubuntu1 (Accepted)".

It could take time before Launchpad builds the binary package and before to be able to install it.

See [Packaging/PPA/Uploading](https://help.launchpad.net/Packaging/PPA/Uploading) for details.

### (13.3) Test installing the package from the PPA

Ubuntu users can install your packages in just the same way they install standard Ubuntu packages and they'll automatically receive updates as and when we make them.

NOTE: **Once the Launchpad build page finally says the package is built, it's finally safe to run the usual 'apt-get update'**.

Example:

```sh
sudo add-apt-repository ppa:apaz/test
sudo apt update
```

See [Packaging/PPA/InstallingSoftware](https://help.launchpad.net/Packaging/PPA/InstallingSoftware) for details.

### How to remove a PPA repository

Example:

```sh
sudo add-apt-repository --remove ppa:apaz/test
sudo apt update
```

## (14) Publishing into a PPA on Launchpad - Alternative 2 - Create the source package using a recipe (aka. daily build)

NOTE: Before to proceed, we should have confirmed that our build works by testing it locally.

The recipe is a simple description of what steps are needed to construct a package from Bazaar or Git branches. It specifies:
- which branch to use to extract the source code
- which branch to use to find the packaging information
- which version to give the package
- what to modify to make the source build properly

To create a source package recipe, we need:
- a repository in Launchpad (either hosted directly on Launchpad or imported from elsewhere)
- a branch (it can be either a Bazaar or Git branch) with buildable code
- a recipe

NOTE: Source package builds is the name of this Launchpad feature. Most source package builds are automatically built each day and we'll see those referred to as "daily builds" because it allows to get an automatic build on every day the source changes, which is then published in the corresponding PPA.

See [Packaging/SourceBuilds](https://help.launchpad.net/Packaging/SourceBuilds), [Packaging/SourceBuilds/GettingStarted](https://help.launchpad.net/Packaging/SourceBuilds/GettingStarted) and [Packaging/SourceBuilds/Recipes](https://help.launchpad.net/Packaging/SourceBuilds/Recipes) for details.

### (14.1) Publish the code on Launchpad

Using Launchpad, we can publish Bazaar branches or Git repositories of your code and, optionally, associate them with [Launchpad's projects](https://help.launchpad.net/Projects/Registering). You can also mirror Bazaar branches that are hosted elsewhere on the internet and even import git, Subversion and CVS repositories into Bazaar branches.

**We do not need to register a Launchpad's project to publish into a PPA**, so here we will proceed by uploading our changes into a "personal" repository with no particular connection to any project or package (like "+junk" in Launchpad's Bazaar code hosting).

In the following steps we will upload our code in a personal git repository. We can see our git repositories at https://code.launchpad.net/~OWNER/+git, for example [https://code.launchpad.net/~apaz/+git](https://code.launchpad.net/~apaz/+git).

See [Code/Git](https://help.launchpad.net/Code/Git) for details about managing git repositories on Launchpad.

#### (14.1.1) Maintain a Debian package in git with 'git-buildpackage' (gbp)

NOTE: It is not mandatory to use git-buildpackage, but the tool is easy to use and also used for CI/CD.

DEP-14 describes a convention for branch layout that can be maintained with 'gbp'. This specification basically has the following principles:
- Each "vendor" uses its own namespace for its packaging related Git branches and tags: 'debian/\*' for Debian, 'ubuntu/\*' for Ubuntu, and so on.
- Packages uploaded to the current development release should be prepared in either a '\<vendor\>/latest' or '\<vendor\>/\<suite\>' branch, where '\<suite\>' is the suite name of the target distribution. Coexistence of '\<vendor\>/latest' and '\<vendor\>/\<suite\>' branches is accepted only if the latter are short-lived, i.e. they exist only until they are merged into '\<vendor\>/latest' or until the version in the target distribution is replaced by the version in '\<vendor\>/latest'. In Debian this means that uploads to unstable and experimental should be prepared either in the 'debian/latest' branch or respectively in the 'debian/unstable' and 'debian/experimental' branches.
- If the Git workflow in use imports the upstream sources from released tarballs, this should be done under the "upstream" namespace. By default, the latest upstream version should be imported in the 'upstream/latest' branch and when packages for multiple upstream versions are maintained concurrently, one should create as many upstream branches as required. Their name should be based on the major upstream version tracked: for example when upstream maintains a stable 1.2 branch and releases 1.2.x minor releases in that branch, those releases should be imported in a 'upstream/1.2.x' branch (the ".x" suffix makes it clear that we are referring to a branch and not to the tag corresponding the upstream 1.2 release). If the upstream developers use codenames to refer to their releases, the upstream branches can be named according to those codenames.

One of the advantages of the DEP-14 branch layout is that it easily allows mixing upstream development and Debian patching in the same repository without mixing Debian packaging files with upstream files.

In this example we will be creating a git repository and importing the uptream tarball by hand:

```sh
# create a git repo for both upstream sources and packaging information
mkdir ../hello
cd $_

git init

gbp import-orig \
  --debian-branch=debian/latest \
  --upstream-branch=upstream/latest \
  ../hello_2.12.orig.tar.gz
# What will be the source package name? [hello]
# What is the upstream version? [2.12]
# gbp:info: Importing '../hello_2.12.orig.tar.gz' to branch 'upstream/latest'...
# gbp:info: Source package is hello
# gbp:info: Upstream version is 2.12
# gbp:info: Successfully imported version 2.12 of ../hello_2.12.orig.tar.gz



# checking
git branch
# * debian/latest
#   upstream/latest
git tag
# upstream/2.12
ls
# ABOUT-NLS   ChangeLog.O   COPYING      lib          man         src
# aclocal.m4  config.in     doc          m4           NEWS        tests
# AUTHORS     configure     GNUmakefile  maint.mk     po          THANKS
# build-aux   configure.ac  hello.1      Makefile.am  README      TODO
# ChangeLog   contrib       INSTALL      Makefile.in  README-dev



# copy debian/ dir
mv ../hello-2.12/debian/ .
git add .
git commit -m "Initial packaging information"
```

There exists another alternative of downloading and import a new upstream version using the information from 'debian/watch' with the option '--uscan'. See `man gbp-import-orig` and [DEP-14](https://dep-team.pages.debian.net/deps/dep14/) for details.

##### Adding entries to 'debian/changelog' using 'gbp'

The command 'gbp dch' generates Debian changelog entries from git commit messages. It means that all commits (not registered in the previous version) will be written in the 'debian/changelog'.

The following example shows examples of:
- using this command without commits since the last commited change of the changelog - this updates the current release entry, similar to `dch -r ""`.
- using this command after few example commits - this adds a new release entry, with new entries (the bullet points, entries with the '*' character in front) for each commit message.

```sh
# Test without commits since last update of 'debian/changelog'
gbp dch \
  --debian-branch=debian/latest
# gbp:info: Changelog last touched at '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Continuing from commit '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: No changes detected from 23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f to HEAD.

git diff
# diff --git a/debian/changelog b/debian/changelog
# index 3d5fcb2..2d7f4e1 100644
# --- a/debian/changelog
# +++ b/debian/changelog
# @@ -1,6 +1,5 @@
# -hello (2.12-0ubuntu1) jammy; urgency=low
# +hello (2.12-0ubuntu1) jammy; urgency=medium
#
#    * Initial packaging.
#
# - -- Aldo Paz <aldo.paz@noemail.org>  Wed, 08 Jun 2022 22:42:27 -0300
# -
# + -- Aldo Paz <aldo.paz@noemail.org>  Thu, 16 Jun 2022 02:29:37 -0300



# Test with example commits
git commit --allow-empty -m "New changes"
# [debian/latest ae29afc] New changes
git commit --allow-empty -m "More changes"
# [debian/latest 8f635d0] More changes

# gbp dch \
#   --debian-branch=debian/latest \
#   --new-version=2.12-0ubuntu2
# or...
gbp dch \
  --debian-branch=debian/latest
# gbp:info: Changelog last touched at '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Continuing from commit '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'

cat debian/changelog
# hello (2.12-0ubuntu2) UNRELEASED; urgency=medium
#
#   * New changes
#   * More changes
#
#  -- Aldo Paz <aldo.paz@noemail.org>  Thu, 16 Jun 2022 02:38:15 -0300
#
# hello (2.12-0ubuntu1) jammy; urgency=low
#
#   * Initial packaging.
#
#  -- Aldo Paz <aldo.paz@noemail.org>  Wed, 08 Jun 2022 22:42:27 -0300
```

As we can see above `gbp dch` increments the version automatically but, if necessary, we could specify the version using the parameter '--new-version=\<version\>'.

##### Git clone, commit, and tag using 'gbp'

We can use commands like 'gbp clone', 'gbp dch --commit', 'gbp tag' (or 'gbp buildpackage --git-tag-only'), 'gbp pull' and 'gbp push' to do common git tasks.

For example we can create a new version and commit everything using 'git dch --commit', then we could tag this commit with the same version of the package using `gbp buildpackage --git-tag-only` (its name will be "debian/X.X").

NOTE: The commands `gbp tag --debian-branch=debian/latest` and `gbp buildpackage --git-tag-only` do the same thing. The later doesn't build, only tag and run post-tag hooks.

Example:

Before to execute 'gbp clone' I had to configure the default branch of the repository in Launchpad. I had to go to https://code.launchpad.net/~apaz/+git/hello', click "Change repository details" and change the "Default branch" value from "refs/heads/master" to "debian/latest", and finally saved the changes clicking "Change Git Repository".

```sh
# Clone the repository and set up tracking branches for the debian (default branch: master) and upstream (default branch: upstream) branches
# options:
#   --all - Track all branches, not only debian and upstream.
#   --debian-branch=branch_name - The branch in the Git repository the Debian package is being developed on, default is master.
#   --upstream-branch=branch_name - The branch in the Git repository the upstream sources are put onto. Default is upstream.
#   --pristine-tar - Track pristine tar branch.
gbp clone \
  --verbose \
  --all \
  --debian-branch=debian/latest \
  --upstream-branch=upstream/latest \
  git+ssh://apaz@git.launchpad.net/~apaz/+git/hello
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:info: Cloning from 'git+ssh://apaz@git.launchpad.net/~apaz/+git/hello'
# gbp:debug: ['git', 'clone', '--quiet', 'git+ssh://apaz@git.launchpad.net/~apaz/+git/hello']
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'for-each-ref', '--format=%(refname:short)', 'refs/remotes/']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/HEAD']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/debian/latest']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/upstream/latest']
# gbp:debug: ['git', 'branch', 'upstream/latest', 'origin/upstream/latest']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/remotes/debian/latest']
# gbp:debug: ['git', 'config', 'user.name', 'Aldo Paz']
# gbp:debug: ['git', 'config', 'user.email', 'aldo.paz@noemail.org']
# gbp:debug: ['git', 'ls-tree', '-z', '-r', '-l', 'HEAD', '--']
cd hello/
git branch -a
# * debian/latest
#   upstream/latest
#   remotes/origin/HEAD -> origin/debian/latest
#   remotes/origin/debian/latest
#   remotes/origin/upstream/latest


# Test 'gbp dch' with example commits
git commit --allow-empty -m "New changes"
# [debian/latest 4b02bdc] New changes
git commit --allow-empty -m "More changes"
# [debian/latest 54891c7] More changes

gbp dch \
  --debian-branch=debian/latest \
  --commit
# gbp:info: Changelog last touched at '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Continuing from commit '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Changelog committed for version 2.12-0ubuntu2


# Check
git log --oneline
# 52b59b6 (HEAD -> debian/latest) Update changelog for 2.12-0ubuntu2 release
# 54891c7 More changes
# 4b02bdc New changes
# 23d11f4 (origin/debian/latest, origin/HEAD) Initial packaging information
# 9d36488 (origin/upstream/latest, upstream/latest) New upstream version 2.12
git tag
# (empty, no tags yet)


# Tag the commit (its name will be "debian/X.X")
# gbp tag --debian-branch=debian/latest
# or
gbp buildpackage \
  --git-debian-branch=debian/latest \
  --git-tag-only
# gbp:info: Tagging Debian package 2.12-0ubuntu2 as debian/2.12-0ubuntu2 in git


# Check
git tag
# debian/2.12-0ubuntu2
git tag -n
# debian/2.12-0ubuntu2 hello Debian release 2.12-0ubuntu2


# Push the tags and their respective revisions (it does not updates the remote branches)
# push the latest tag
#   git push origin debian/2.12-0ubuntu2
# or, push all local tags
#   git push origin --tags
# or, using gbp push...
# options:
#   --debian-branch=branch_name - The branch in the Git repository the Debian package is being developed on. If set to the empty value the branch will not be pushed.
#   --debian-tag=TAG-FORMAT - Use this tag format when looking for tags corresponding to a Debian version. Default is debian/%(version)s. If set to the empty value the tag will not be pushed.
#   --upstream-branch=branch_name - The branch in the Git repository the upstream sources are put onto. If set to the empty value the branch will not be pushed.
#   --upstream-tag=TAG-FORMAT - Use this tag format when looking for tags of upstream versions. Default is upstream/%(version)s. If set to the empty value the tag will not be pushed.
#   --pristine-tar - Whether to update the pristine-tar branch too.
gbp push \
  --verbose \
  --debian-branch=debian/latest \
  --upstream-branch=
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'symbolic-ref', 'HEAD']
# gbp:debug: ['git', 'show-ref', 'refs/heads/debian/latest']
# gbp:debug: ['git', 'config', 'branch.debian/latest.remote']
# gbp:debug: ['git', 'config', 'branch.debian/latest.merge']
# gbp:debug: ['git', 'tag', '-l', 'debian/2.12-0ubuntu2']
# gbp:debug: ['git', 'tag', '-l', 'upstream/2.12']
# gbp:info: Pushing debian/2.12-0ubuntu2 to origin
# gbp:debug: ['git', 'push', 'origin', 'tag', 'debian/2.12-0ubuntu2']
```

Instead of using 'gbp', the tag can be done manually, the equivalent is `git tag -a debian/X.X -m "Update for X.X release"`. For this example, the equivalent command would be `git tag -a debian/2.12-0ubuntu2 -m "hello Debian release 2.12-0ubuntu2"`.

See `man gbp-clone`, `man gbp-dch`, `man gbp-tag`, `man gbp-buildpackage`, `man gbp-pull` and `man gbp-push` for details.

#### (14.1.3) Test building the package with 'gbp'

```sh
gbp buildpackage \
  --git-debian-branch=debian/latest \
  --git-export-dir=../build-area
#...

# checking
ls ../build-area/
# hello_2.12-0ubuntu1_amd64.build      hello_2.12-0ubuntu1.debian.tar.xz
# hello_2.12-0ubuntu1_amd64.buildinfo  hello_2.12-0ubuntu1.dsc
# hello_2.12-0ubuntu1_amd64.changes    hello_2.12.orig.tar.gz
# hello_2.12-0ubuntu1_amd64.deb        hello-dbgsym_2.12-0ubuntu1_amd64.ddeb

debc ../build-area/hello_2.12-0ubuntu1_amd64.changes
# ...
```

We could add the option '--git-ignore-new' if we want to build the package even if some changes have not been commited. See `man gbp-buildpackage` for details.

#### (14.1.2) Push into a personal respository on Launchpad

```sh
# git remote add launchpad git+ssh://USER@git.launchpad.net/~OWNER/+git/REPOSITORY

git remote add launchpad git+ssh://apaz@git.launchpad.net/~apaz/+git/hello
git push launchpad --all
# Enumerating objects: 473, done.
# Counting objects: 100% (473/473), done.
# Delta compression using up to 4 threads
# Compressing objects: 100% (468/468), done.
# Writing objects: 100% (473/473), 1.09 MiB | 268.00 KiB/s, done.
# Total 473 (delta 128), reused 0 (delta 0), pack-reused 0
# remote: Resolving deltas: 100% (128/128), done.
# To git+ssh://git.launchpad.net/~apaz/+git/hello
#  * [new branch]      debian/latest -> debian/latest
#  * [new branch]      upstream/latest -> upstream/latest
```

With the 'push' command above, the new repository should be accessible, for example, on 'https://code.launchpad.net/~apaz/+git/hello' and the corresponding cgit page on 'https://git.launchpad.net/~apaz/+git/hello'.

##### Defining an abbreviation for repositories hosted on Launchpad

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

##### Repository URLs

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

### (14.2) Write the recipe

A "recipe" in Launchpad is a description of the steps needed to construct a package from a set of Bazaar or Git branches. It allows to automatize the packaging and then host the builded packages.

The format of a recipe specifies:

- Which branch to use for the _source code_: trunk branch, beta branch, etc.
- Which branch to use for the _packaging information_: separate branch, Ubuntu branch, etc.
- The correct _package version_ (so people can still upgrade to the new stable version when it's released).
- What to modify to make the source build properly.

There are different ways to write a recipe. The following is the most simple example, and will be the code for testing our repository build on Launchpad:

```
# git-build-recipe format 0.4 deb-version {debupstream}~{revno}
lp:~apaz/+git/hello debian/latest
```

I saved the file as 'hello.recipe'. It assumes both the source code and the packaging information (the debian/ directory) exists in the same directory.

See [Packaging/SourceBuilds/GettingStarted](https://help.launchpad.net/Packaging/SourceBuilds/GettingStarted) and [Packaging/SourceBuilds/Recipes](https://help.launchpad.net/Packaging/SourceBuilds/Recipes) for details.

#### Example recipe for merging two different repositories/branches

For the following example we have to write a recipe that:
- use the project's trunk, which contains no packaging,
- and nest another branch that contains only packaging information.

```
# git-build-recipe format 0.4 deb-version {debupstream}~{revno}
lp:~apaz/+git/hello
merge packaging lp:~apaz/+git/hello-packaging
```

Where:
- The first line tells Launchpad which recipe version we're using (in this case it's 0.3), along with how we want to name the resultant package.
- The next line specifies the code branch, using Launchpad's short name system.
- Finally, we specify where to find the packaging branch and say that we want to nest it into the code branch.

### (14.3) Install 'git-build-recipe' and test the recipe

NOTE: We should always test the recipe locally before sending it to Launchpad.

We need 'git-build-recipe' to test the recipe locally:

```sh
sudo apt-get install git-build-recipe
```

The following processes the recipe and creates, inside the current directory, a directory called 'working-dir', into which it places the resulting source tree and source package:

```sh
cd ..

vi hello.recipe
# ... write the code of the recipe

git-build-recipe --allow-fallback-to-native hello.recipe recipe-working-dir
# ...
```

### (14.4) Create the recipe on Launchpad

Browse to the branch you want to build in Launchpad and click "(+) Create packaging recipe".

We have to configure the recipe writing the code of the recipe we tested before in the 'Recipe text:' field. Finaly click on 'Create Recipe'.

For our example, the above steps created the recipe on [https://code.launchpad.net/~apaz/+recipe/hello-daily](https://code.launchpad.net/~apaz/+recipe/hello-daily).

We could select "Build Now" in order to schedule a new build quickly.

## Packaging using a Debian uscan file ('debian/watch')

This file allows Debian QA tools to alert for new upstream releases and also git-buildpackage to easily download the new upstream releases.

The file named watch in the debian directory is used to check for newer versions of upstream software is available and to download it if necessary. The download itself will be performed with the 'uscan' program from the 'devscripts' package.

For this example, 'debian/watch' will be:

```sh
version=4

http://ftp.gnu.org/gnu/hello/hello-(.*).tar.gz

```

The commands below shows how to run 'uscan' for downloading the version of upstream specified on 'debian/changelog' and the latest:

```sh
# create a new repository dir
mkdir hello-packaging
cd $_



# copy the packaging information (debian/ directory) into the new repository
cp -R ../hello-packaging-old/debian .

# create the 'watch' file
vi debian/watch
#...



# test it
mkdir ../build-dir
uscan --no-download --verbose --destdir ../build-dir
# uscan info: uscan (version 2.22.1ubuntu1) See uscan(1) for help
# uscan info: Scan watch files in .
# uscan info: Check debian/watch and debian/changelog in .
# uscan info: package="hello" version="2.12-0ubuntu1" (as seen in debian/changelog)
# uscan info: package="hello" version="2.12" (no epoch/revision)
# uscan info: ./debian/changelog sets package="hello" version="2.12"
# uscan info: Process watch file at: debian/watch
#     package = hello
#     version = 2.12
#     pkg_dir = .
# uscan info: Last orig.tar.* tarball version (from debian/changelog): 2.12
# uscan info: Last orig.tar.* tarball version (dversionmangled): 2.12
# uscan info: Requesting URL:
#    http://ftp.gnu.org/gnu/hello/
# uscan info: Matching pattern:
#    (?:(?:http://ftp.gnu.org)?\/gnu\/hello\/)?hello-(.*).tar.gz
# uscan info: Found the following matching hrefs on the web page (newest first):
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
# uscan info: Looking at $base = http://ftp.gnu.org/gnu/hello/ with
#     $filepattern = hello-(.*).tar.gz found
#     $newfile     = http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
#     $newversion  = 2.12.1
#     $lastversion = 2.12
# uscan info: Matching target for downloadurlmangle: http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Upstream URL(+tag) to download is identified as    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Filename (filenamemangled) for downloaded file: hello-2.12.1.tar.gz
# uscan: Newest version of hello on remote site is 2.12.1, local version is 2.12
# uscan:  => Newer package available from:
#         => http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Scan finished



# Download the latest version
uscan --verbose --destdir ../build-dir
# uscan info: uscan (version 2.22.1ubuntu1) See uscan(1) for help
# uscan info: Scan watch files in .
# uscan info: Check debian/watch and debian/changelog in .
# uscan info: package="hello" version="2.12-0ubuntu1" (as seen in debian/changelog)
# uscan info: package="hello" version="2.12" (no epoch/revision)
# uscan info: ./debian/changelog sets package="hello" version="2.12"
# uscan info: Process watch file at: debian/watch
#     package = hello
#     version = 2.12
#     pkg_dir = .
# uscan info: Last orig.tar.* tarball version (from debian/changelog): 2.12
# uscan info: Last orig.tar.* tarball version (dversionmangled): 2.12
# uscan info: Requesting URL:
#    http://ftp.gnu.org/gnu/hello/
# uscan info: Matching pattern:
#    (?:(?:http://ftp.gnu.org)?\/gnu\/hello\/)?hello-(.*).tar.gz
# uscan info: Found the following matching hrefs on the web page (newest first):
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
# uscan info: Looking at $base = http://ftp.gnu.org/gnu/hello/ with
#     $filepattern = hello-(.*).tar.gz found
#     $newfile     = http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
#     $newversion  = 2.12.1
#     $lastversion = 2.12
# uscan info: Matching target for downloadurlmangle: http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Upstream URL(+tag) to download is identified as    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Filename (filenamemangled) for downloaded file: hello-2.12.1.tar.gz
# uscan: Newest version of hello on remote site is 2.12.1, local version is 2.12
# uscan:  => Newer package available from:
#         => http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
# uscan info: Not downloading, using existing file: hello-2.12.1.tar.gz
# uscan info: Start checking for common possible upstream OpenPGP signature files
# uscan warn: Possible OpenPGP signature found at:
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
#  * Add opts=pgpsigurlmangle=s/$/.sig/ or opts=pgpmode=auto to debian/watch
#  * Add debian/upstream/signing-key.asc.
#  See uscan(1) for more details
# uscan info: End checking for common possible upstream OpenPGP signature files
# uscan info: Missing OpenPGP signature.
# uscan info: New orig.tar.* tarball version (oversionmangled): 2.12.1
# uscan info: Launch mk-origtargz with options:
#    --package hello --version 2.12.1 --compression default --directory ../build-dir --copyright-file debian/copyright ../build-dir/hello-2.12.1.tar.gz
# Leaving ../build-dir/hello_2.12.1.orig.tar.gz where it is.
# uscan info: New orig.tar.* tarball version (after mk-origtargz): 2.12.1
# uscan info: Scan finished



# Download the current version
uscan --download-current-version --verbose --destdir ../build-dir
# uscan info: uscan (version 2.22.1ubuntu1) See uscan(1) for help
# uscan info: Scan watch files in .
# uscan info: Check debian/watch and debian/changelog in .
# uscan info: package="hello" version="2.12-0ubuntu1" (as seen in debian/changelog)
# uscan info: package="hello" version="2.12" (no epoch/revision)
# uscan info: ./debian/changelog sets package="hello" version="2.12"
# uscan info: Process watch file at: debian/watch
#     package = hello
#     version = 2.12
#     pkg_dir = .
# uscan info: Last orig.tar.* tarball version (from debian/changelog): 2.12
# uscan info: Download the --download-current-version specified version: 2.12
# uscan info: Requesting URL:
#    http://ftp.gnu.org/gnu/hello/
# uscan info: Matching pattern:
#    (?:(?:http://ftp.gnu.org)?\/gnu\/hello\/)?hello-(.*).tar.gz
# uscan info: Found the following matching hrefs on the web page (newest first):
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz (2.12.1) index=2.12.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 matched with the download version
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz (2.12) index=2.12-1 matched with the download version
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.11.tar.gz (2.11) index=2.11-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz (2.10) index=2.10-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.9.tar.gz (2.9) index=2.9-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.8.tar.gz (2.8) index=2.8-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.7.tar.gz (2.7) index=2.7-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.6.tar.gz (2.6) index=2.6-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.5.tar.gz (2.5) index=2.5-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.4.tar.gz (2.4) index=2.4-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.3.tar.gz (2.3) index=2.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.2.tar.gz (2.2) index=2.2-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.1.tar.gz (2.1.1) index=2.1.1-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-2.1.0.tar.gz (2.1.0) index=2.1.0-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
#    http://ftp.gnu.org/gnu/hello/hello-1.3.tar.gz (1.3) index=1.3-1 
# uscan info: Looking at $base = http://ftp.gnu.org/gnu/hello/ with
#     $filepattern = hello-(.*).tar.gz found
#     $newfile     = http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz
#     $newversion  = 2.12
#     $lastversion = 2.12
# uscan info: Matching target for downloadurlmangle: http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz
# uscan info: Upstream URL(+tag) to download is identified as    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz
# uscan info: Filename (filenamemangled) for downloaded file: hello-2.12.tar.gz
# uscan: Newest version of hello on remote site is 2.12, specified download version is 2.12
# uscan info: Downloading upstream package: hello-2.12.tar.gz
# uscan info: Requesting URL:
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz
# uscan info: Successfully downloaded upstream package: hello-2.12.tar.gz
# uscan info: Start checking for common possible upstream OpenPGP signature files
# uscan warn: Possible OpenPGP signature found at:
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz.sig
#  * Add opts=pgpsigurlmangle=s/$/.sig/ or opts=pgpmode=auto to debian/watch
#  * Add debian/upstream/signing-key.asc.
#  See uscan(1) for more details
# uscan info: End checking for common possible upstream OpenPGP signature files
# uscan info: Missing OpenPGP signature.
# uscan info: New orig.tar.* tarball version (oversionmangled): 2.12
# uscan info: Launch mk-origtargz with options:
#    --package hello --version 2.12 --compression default --directory ../build-dir --copyright-file debian/copyright ../build-dir/hello-2.12.tar.gz
# Successfully symlinked ../build-dir/hello-2.12.tar.gz to ../build-dir/hello_2.12.orig.tar.gz.
# uscan info: New orig.tar.* tarball version (after mk-origtargz): 2.12
# uscan info: Scan finished



# Check what was downloaded
ls ../build-dir/
# hello_2.12.1.orig.tar.gz -> hello-2.12.1.tar.gz
# hello-2.12.1.tar.gz
# hello_2.12.orig.tar.gz -> hello-2.12.tar.gz
# hello-2.12.tar.gz
```

See `man uscan` and [debian/watch in debian wiki](https://wiki.debian.org/debian/watch) for details.

### Testing and troubleshooting with 'uscan'

To test the debian/watch implementation we can execute:

```sh
  uscan --no-download --verbose
```

Also, if 'uscan' is not working as expected, we can use '--debug' to see what it's fetching and what it's (not) matching:

```sh
uscan --no-download --verbose --debug
```

### Add a new upstream release to a repository using 'gbp' and 'uscan'

'gbp import-orig' already includes an integration to use 'uscan' using the '--uscan' option.

Example before and after adding 'debian/watch':

```sh
gbp clone \
  --verbose \
  --all \
  --debian-branch=debian/latest \
  --upstream-branch=upstream/latest \
  git+ssh://apaz@git.launchpad.net/~apaz/+git/hello
# ...

cd hello/
git checkout debian/latest


gbp import-orig \
  --verbose \
  --uscan \
  --debian-branch=debian/latest \
  --upstream-branch=upstream/latest
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'for-each-ref', '--format=%(refname:short)', 'refs/heads/']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/upstream/latest']
# gbp:debug: ['git', 'status', '--porcelain']
# gbp:info: Launching uscan...
# uscan warn: No watch file found
# gbp:error: Uscan failed: No watch file found


vi debian/watch
# adding ...
# version=4
#
# http://ftp.gnu.org/gnu/hello/hello-(.*).tar.gz
git add .
git commit -m "Add debian/watch"


gbp dch \
  --debian-branch=debian/latest \
  --commit
# gbp:info: Changelog last touched at '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Continuing from commit '23d11f4a7b6e695bbdcacd7e2ffa8d5fdcf9957f'
# gbp:info: Changelog committed for version 2.12-0ubuntu2


head -10 debian/changelog
# hello (2.12-0ubuntu2) UNRELEASED; urgency=medium
#
#   * Add debian/watch
#
#  -- Aldo Paz <aldo.paz@noemail.org>  Fri, 24 Jun 2022 02:51:53 -0300
#
# hello (2.12-0ubuntu1) jammy; urgency=low
#
#   * Initial packaging.


gbp import-orig \
  --verbose \
  --uscan \
  --debian-branch=debian/latest \
  --upstream-branch=upstream/latest
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'for-each-ref', '--format=%(refname:short)', 'refs/heads/']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/upstream/latest']
# gbp:debug: ['git', 'status', '--porcelain']
# gbp:info: Launching uscan...
# uscan warn: Possible OpenPGP signature found at:
#    http://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
#  * Add opts=pgpsigurlmangle=s/$/.sig/ or opts=pgpmode=auto to debian/watch
#  * Add debian/upstream/signing-key.asc.
#  See uscan(1) for more details
# gbp:info: Using uscan downloaded tarball ../hello_2.12.1.orig.tar.gz
# What is the upstream version? [2.12.1] 
# gbp:debug: ['git', 'tag', '-l', 'upstream/2.12.1']
# gbp:debug: tar ['-C', '../tmp8ckfk7lk', '-a', '-xf', '../hello_2.12.1.orig.tar.gz'] []
# gbp:debug: Unpacked '../hello_2.12.1.orig.tar.gz' to '../tmp8ckfk7lk/hello-2.12.1'
# gbp:info: <DebianUpstreamSource path='../hello_2.12.1.orig.tar.gz' signaturefile=None>
# gbp:info: Importing '../hello_2.12.1.orig.tar.gz' to branch 'upstream/latest'...
# gbp:info: Source package is hello
# gbp:info: Upstream version is 2.12.1
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/upstream/latest']
# gbp:debug: ['git', 'rev-parse', '--quiet', '--verify', 'upstream/latest']
# gbp:debug: ['git', 'add', '-f', '.']
# gbp:debug: ['git', 'write-tree']
# gbp:debug: ['git', 'rev-parse', '--quiet', '--verify', 'upstream/latest']
# gbp:debug: ['git', 'commit-tree', 'ad5fc7c3062e8426b7936588e7a27d51ace0e508', '-p', '9d36488c717869697816fe3bb6d711f93aee0a78']
# gbp:debug: ['git', 'update-ref', '-m', 'gbp: New upstream version 2.12.1', 'refs/heads/upstream/latest', 'a03d034d9f6e55762c88ab985c40af543b7c8b0a', '9d36488c717869697816fe3bb6d711f93aee0a78']
# gbp:debug: ['git', 'tag', '-m', 'Upstream version 2.12.1', 'upstream/2.12.1', 'a03d034d9f6e55762c88ab985c40af543b7c8b0a']
# gbp:debug: ['git', 'show-ref', '--verify', 'refs/heads/debian/latest']
# gbp:debug: ['git', 'rev-parse', '--quiet', '--verify', 'debian/latest']
# gbp:debug: ['git', 'show', '--pretty=medium', 'debian/latest:debian/source/format']
# gbp:debug: 3.0 (quilt) package, replacing debian/ dir
# gbp:info: Replacing upstream source on 'debian/latest'
# gbp:debug: ['git', 'ls-tree', '-z', 'upstream/2.12.1^{tree}', '--']
# gbp:debug: ['git', 'ls-tree', '-z', 'debian/latest^{tree}', '--']
# gbp:debug: Using 8f561ccc30064b19e3bdca76e7f4e6baaed89516 as debian/ tree
# gbp:debug: ['git', 'mktree', '-z']
# gbp:debug: ['git', 'commit-tree', '74fe57634d12f5016f9c1d5dc5c428343891f330', '-p', 'debian/latest^{commit}', '-p', 'upstream/2.12.1^{commit}']
# gbp:debug: ['git', 'update-ref', '-m', 'gbp: Updating debian/latest after import of upstream/2.12.1', 'refs/heads/debian/latest', '304224fa3b18e8f8409723dce0f7220aab8e4e8f']
# gbp:debug: ['git', 'symbolic-ref', 'HEAD']
# gbp:debug: ['git', 'show-ref', 'refs/heads/debian/latest']
# gbp:debug: ['git', 'reset', '--quiet', '--hard', '304224fa3b18e8f8409723dce0f7220aab8e4e8f', '--']
# gbp:debug: ['git', 'symbolic-ref', 'HEAD']
# gbp:debug: ['git', 'show-ref', 'refs/heads/debian/latest']
# gbp:debug: rm ['-rf', '../tmp8ckfk7lk'] []
# gbp:info: Successfully imported version 2.12.1 of ../hello_2.12.1.orig.tar.gz


ls ..
# hello/
# hello_2.12.1.orig.tar.gz -> hello-2.12.1.tar.gz
# hello-2.12.1.tar.gz


git tag
# debian/2.12-0ubuntu2
# upstream/2.12.1
git tag -n
# debian/2.12-0ubuntu2 hello Debian release 2.12-0ubuntu2
# upstream/2.12.1 Upstream version 2.12.1


git log --oneline --graph
# *   304224f (HEAD -> debian/latest) Update upstream source from tag 'upstream/2.12.1'
# |\  
# | * a03d034 (tag: upstream/2.12.1, upstream/latest) New upstream version 2.12.1
# * | ab58190 Update changelog for 2.12-0ubuntu2 release
# * | 36281a5 Add debian/watch
# * | 23d11f4 (origin/debian/latest, origin/HEAD) Initial packaging information
# |/  
# * 9d36488 (origin/upstream/latest) New upstream version 2.12


# Push the tags
gbp push \
  --verbose \
  --debian-branch=debian/latest \
  --upstream-branch=
# gbp:debug: ['git', 'rev-parse', '--show-cdup']
# gbp:debug: ['git', 'rev-parse', '--is-bare-repository']
# gbp:debug: ['git', 'rev-parse', '--git-dir']
# gbp:debug: ['git', 'symbolic-ref', 'HEAD']
# gbp:debug: ['git', 'show-ref', 'refs/heads/debian/latest']
# gbp:debug: ['git', 'config', 'branch.debian/latest.remote']
# gbp:debug: ['git', 'config', 'branch.debian/latest.merge']
# gbp:debug: ['git', 'tag', '-l', 'debian/2.12-0ubuntu2']
# gbp:debug: ['git', 'tag', '-l', 'upstream/2.12']
# gbp:info: Pushing debian/2.12-0ubuntu2 to origin
# gbp:debug: ['git', 'push', 'origin', 'tag', 'debian/2.12-0ubuntu2']
```

## Layouts for Git packaging repositories

There are two different repository layouts, that can be used by package maintainers. Both layouts can be easily handled by 'git-buildpackage' (gbp).

### DEP-14 style or prestine-tar layout

Both the Upstream sources and the packaging files are stored in Git. This means the repository include the full source code of the upstream project.

In this usage case, the upstream source code is imported under branch upstream, or even the original upstream GIT repository is used as base, with its branches like 'master'. When importing upstream releases, GIT changes will only show a single commit for the version, with added GIT tag.

To have the full file tree, there also will be a 'debian/master' or 'master' branch, which includes the debian/ directory. Upstream changes are then merged to the Debian packaging branch.

As a result, after a GIT clone, you can just start building, and all file changes can be diffed against GIT.

As optional add-on you often find a 'pristine-tar' branch (this means the maintainers are using 'dbp' with 'prestine-tar'), this branch is used to store metadata, so the original tarball can be recreated from the GIT branches.

### The overlay layout

Aka. debian-only or simple layout.

Only the packaging files (only the debian/ directory) is stored in Git, plus maybe some CI and metadata to build with. It is easier to handle in terms of repository sizes but you will need to take care of downloading and extracting the upstream's source code with tools like 'origtargz', 'uscan' or similars.

In this mode you should find a very simple 'master' or 'debian/master' branch.


## Configuring 'git-buildpackage' (gbp)

The configuration can be done in several files:
- '/etc/git-buildpackage/gbp.conf' - system-wide.
- '~/.gbp.conf' - per user.
- 'debian/gbp.conf' - per repository, distributable.
- '.git/gbp.conf' - per repository, not distributed.

If you need a skeleton, just copy '/etc/git-buildpackage/gbp.conf' to any of the above locations. The options are explained in `man 5 gbp.conf` too.

### Common configuration for prestine-tar layout

```ini
[DEFAULT]
builder = ...
pristine-tar = true

[buildpackage]
sign-tags = true
keyid = <key-id>
export-dir = /tmp/build-area/
notify = off

[import-orig]
filter-pristine-tar = true
```

### Common configuration for overlay layout

Because the Git repository will not store the upstream sources in this layout, git-buildpackage needs to be told, where to find the tarball. This can be done by setting the tarball variable in '~/.gbp.conf' to the place, where you store the tarballs. The default value is ../tarballs/. Adjust the path to your tarball location.

Common gbp.conf:

```ini
[DEFAULT]
pristine-tar = false
debian-branch = master
verbose = true

[buildpackage]
overlay = true
```

~/.gbp.conf:

```ini
[buildpackage]
tarball-dir = ~/.local/src/tarballs/
```

## Renaming the orig tarball properly using 'mk-origtargz'

'mk-origtargz' renames the given file to match what is expected by 'dpkg-buildpackage', based on the source package name and version in 'debian/changelog'.

It can convert zip to tar, optionally change the compression scheme and remove files according to 'Files-Excluded' and 'Files-Excluded-component' in 'debian/copyright'. The resulting file is placed in 'debian/../..'.

Example:

```sh
# testing on empty directory
ls
mk-origtargz ../hello-2.12.tar.gz
# mk-origtargz: error: cannot read debian/changelog: No such file or directory

cd ../hello
ls
# debian
head -10 debian/changelog
# hello (2.12-0ubuntu1) jammy; urgency=low
#
#   * Initial packaging.
#
#  -- Aldo Paz <aldo.paz@noemail.org>  Wed, 08 Jun 2022 22:42:27 -0300
#
mk-origtargz ../hello-2.12.tar.gz
# Successfully symlinked ../hello-2.12.tar.gz to ../hello_2.12.orig.tar.gz.
ls ..
# hello
# hello_2.12.orig.tar.gz -> hello-2.12.tar.gz
# hello-2.12.tar.gz
# empty-dir
```

See `man mk-origtargz` for details.

## Downloading the orig tarball of a Debian package from various sources using 'origtargz'

'origtargz' is a tool that downloads the orig tarball of a Debian package (and also can unpack it) into the current directory, if it just contains a debian/ directory. The main use for 'origtargz' is with debian-dir-only repository checkouts (overlay layout), but it is useful as a general tarball download wrapper.

NOTE: 'origtargz' is useful for downloading the current tarball. For Debian package repositories that keep the full upstream source, other tools should be used to upgrade the repository from the new tarball. See `man gbp-import-orig` for examples.

The version number for the tarball to be downloaded is determined from 'debian/changelog'. It should be invoked from the top level directory of an unpacked Debian source package (where the debian/ directory is located).

Various download locations are tried:

- First, an existing file is looked for.
- Directories given with '--path' are searched.
- `pristine-tar` is tried.
- `pristine-lfs` is tried.
- `apt-get source` is tried when `apt-cache showsrc` reports a matching version.
- Finally, `uscan --download --download-current-version` is tried.

The common command used here is `origtargz -dt` that only downloads the orig tarball without unpacking and without downloading the '.dsc', '.diff.gz' and '.debian.tar.gz' components if `apt-get source` is executed.

Example:

```sh
cd hello
ls
# debian

origtargz -dt
# E: You must put some 'deb-src' URIs in your sources.list
# Trying uscan --download --download-current-version ...
# uscan: Newest version of hello on remote site is 2.12, specified download version is 2.12
# uscan warn: Possible OpenPGP signature found at:
#    http://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz.sig
#  * Add opts=pgpsigurlmangle=s/$/.sig/ or opts=pgpmode=auto to debian/watch
#  * Add debian/upstream/signing-key.asc.
#  See uscan(1) for more details
# Successfully renamed ../hello-2.12.tar.gz to ../hello_2.12.orig.tar.gz.

ls
# debian
ls ..
# hello  hello_2.12.orig.tar.gz
```

See `man origtargz` for details.

## About Launchpad's builders

Launchpad builds packages using a slightly modified version of 'buildd' called 'launchpad-buildd'. The corresponding project is located on [https://launchpad.net/launchpad-buildd](https://launchpad.net/launchpad-buildd).

We can get the version of the packages running on Launchpad's builders, including 'launchpad-buildd', from the [buildd PPA](https://launchpad.net/~canonical-is-sa/+archive/ubuntu/buildd).

[The Launchpad build farm](https://launchpad.net/builders) has the list of builders and their current status.

## How to set up 'pbuilder'

'pbuilder' is a tool to do reproducible builds of a package in a clean and isolated environment. In other words, it allows you to build packages locally on your machine ensuring the build will work everywhere and that it's not dependent on something unusual in your own environment.

It serves a couple of purposes:
- The build will be done in a minimal and clean environment. This helps you make sure your builds succeed in a reproducible way, but without modifying your local system.
- There is no need to install all necessary build dependencies locally.
- You can set up multiple instances for various Ubuntu and Debian releases.

```sh
sudo apt install pbuilder

# pbuilder-dist <release> create
pbuilder-dist jammy create
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
