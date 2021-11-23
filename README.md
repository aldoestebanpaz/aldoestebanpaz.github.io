# Aldo's personal site

All the posts in this site are notes and records to help me to remember steps, commands and concepts regarding multiple technologies I've learned at some point. If something here was useful for you, and you feel you can contribute with some addition or fix, please do not hesitate on submitting an issue or making a PR.

## Installation

### Install RVM (steps for Ubuntu)

The following steps applys for Ubuntu only. For other repositories follow the
[official guide](http://rvm.io/rvm/install).

1. Add the PPA:
```sh
sudo apt-add-repository -y ppa:rael-gc/rvm && sudo apt-get update
```
3. Install the RVM package:
```sh
sudo apt-get install rvm
```
4. Add your user to rvm group:
```sh
sudo usermod -a -G rvm $USER
```
5. Close the user session, and open it again.
6. Source the profile script.
```sh
source /etc/profile
```

### Install and use Ruby uwgin RVM

1. Install and use a ruby version:
```sh
rvm install 2.7.2
```
2. Use the installed ruby version:
```sh
rvm use 2.7.2
```

### Install Jekyll

1. Install Jekyll and Bundler

```sh
gem install jekyll bundler
```

## Create a github pages site

Follow the [Creating a GitHub Pages site with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) guide.

## Clone and use the github pages repo

1. Clone the repo:
```sh
git clone git@github.com:aldoestebanpaz/aldoestebanpaz.github.io.git
```
1. Move to repo directory:
```sh
cd aldoestebanpaz.github.io
```
3. Install the dependency packages:
```sh
bundle install
```
4. Run the Jekyll site locally:
```sh
bundle exec jekyll serve
```

## Troubleshooting

### Updating the GitHub Pages gem

Jekyll is an active open source project that is updated frequently. If the github-pages gem on your computer is out of date with the github-pages gem on the GitHub Pages server, your site may look different when built locally than when published on GitHub. To avoid this, regularly update the github-pages gem on your computer.

```sh
bundle update github-pages
```

### Useful RVM commands

- Get usage information of a RVM command: `rvm help COMMAND`.
- List of currently installed ruby strings: `rvm list`.
- List a partial number of valid ruby strings: `rvm list known`.

## References

- [RVM install](https://rvm.io/rvm/install)
- [RVM package for Ubuntu](https://github.com/rvm/ubuntu_rvm).
