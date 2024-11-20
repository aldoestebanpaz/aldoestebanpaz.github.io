

## Using Debuginfod - Ubuntu Jammy (22.04) or later

If you are on Ubuntu Jammy (22.04) or later, you do not need to worry about installing debug symbol packages anymore. The Ubuntu project maintains a Debuginfod server, and GDB and other debuginfo-consumer applications support it out of the box.

Reference:
- https://debuginfod.ubuntu.com/
- https://ubuntu.com/server/docs/service-debuginfod


## Using Debug Symbol Packages

The maintainers of a project usually place the *-dbg.deb packages alongside their binary package in the same repository. Installing them will just involve finding the appropriate package name based on the version of that is installed.

NOTE: Not all maintainers do this and they may not be up to date.

### (1) Enable dbg-/dbgsym- repositories

(1) Create an /etc/apt/sources.list.d/ddebs.list:

```sh
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse
  deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse
  deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" \
  | sudo tee -a /etc/apt/sources.list.d/ddebs.list
```

(2) Then import the debug symbol archive signing key from the Ubuntu server. On Ubuntu 18.04 LTS and newer:

```sh
sudo apt install ubuntu-dbgsym-keyring
```

On earlier releases of Ubuntu use:

```sh
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F2EDC64DC5AEE1F6B9C621F0C8CAB6595FDFF622
```

Note: The GPG key expired on 2021-03-21 and may need updating by either upgrading the ubuntu-dbgsym-keyring package or re-running the apt-key command. Please see Bug #1920640 for workaround details if that does not work.

(3) Then run the following to update your package list or click the Reload button if you used the Synaptic Package Manager.

```sh
sudo apt-get update
```
