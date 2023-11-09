
# Angular usage notes

## Installation

1. Download the latest version:

```sh
curl -OL https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
```

2. Remove any previous Go installation by deleting the /usr/local/go folder (if it exists):

```sh
sudo rm -rf /usr/local/go
```

3. Extract the archive into /usr/local, creating a fresh Go tree in /usr/local/go:

```sh
sudo tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
```

NOTE: Do not untar the archive into an existing /usr/local/go tree. This is known to produce broken Go installations.

4. Add /usr/local/go/bin to the PATH environment variable. You can do this by setting PATH in your $HOME/.profile or /etc/profile (for a system-wide installation) and loading the changes:

```sh
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile
source ~/.profile
```


