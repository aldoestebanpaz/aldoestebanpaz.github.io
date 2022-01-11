# Magento Commerce Cloud local environment setup

The official dev documentation of Magento (now Adobe Commerce) is being updated constantly and could be a little confusing in the beginning because it has a lot of sections, links and generic commaands for reference. That is why I decided to share my notes about the installation process of Magento Commerce Cloud (now Adobe Commerce on cloud) in a local VM for development purposes.

NOTE: In all the examples I'll be using localhost, replace this with the IP or name of your own VM.

## Prepare the linux environment

You have to update the package manager cache with the latest changes of the repositories, and update the system if necessary.

The following commands applies for Centos 7, because that is the system I used when I created this guide.

**Check for updates**

```sh
sudo yum check-update
```

**Update the yum cache**

```sh
sudo yum clean all
sudo yum makecache
```

**Update the essential packages**

```sh
sudo yum update curl nss nss-util nspr
```

## Install main required tools

**Install and configure git**

```sh
sudo yum -y install git
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

**Install php**

```sh
sudo yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y update
# ...
```

TODO: more commands and steps required here. I forgot the commands I've executed in this part.

## Create and switch to the Magento user (aka Magento file system owner)

```sh
## check users in the system
# less /etc/passwd

sudo adduser magento
sudo passwd magento

# give sudo access
sudo usermod -aG wheel magento

# start session as "magento"
su - magento
```

## Install and configure the "Magento Cloud CLI"

### Install

```sh
curl -sS https://accounts.magento.cloud/cli/installer | php
# Executable location will be: /home/magento/.magento-cloud/bin/magento-cloud
source ~/.bash_profile

## check listing "Magento Cloud CLI" commands
# magento-cloud list
```

### Login

**Option 1: using API token**

1. Open https://accounts.magento.cloud/
2. On the Cloud account dashboard, click the Account Settings tab.
3. expand the API Tokens menu.
4. click Create an API Token
5. put an Application name and click Create API Token.
6. copy the generated token.
7. execute following command:
```sh
magento-cloud auth:api-token-login
# put your API Token
```

**Option 2: using a Web-authorization session**

Execute following command:

```sh
magento-cloud login
# go to URL printed on screen for login
```

### Add a SSH key

**Generate the keys**

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
# e.g. ssh-keygen -t ed25519 -C "magento-dev@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Register the SSH into Magento Cloud CLI and check the connection**

```sh
magento-cloud ssh-key:add ~/.ssh/id_ed25519.pub
magento-cloud ssh
```

### Create or download a project

Follow one of these two alternatives.

### Option 1: Create a project

TODO: more commands and steps required here. I forgot the commands I've executed in this part.

### Option 2: Download an existing project

Like git-clone without specifying a directory, the command below will clone eveything inside a directory with the name of the project:

```sh
magento-cloud get <project-id> [<dir-to-clone-to>]
```

## Configure composer

You need to store your public and private keys in a auth.json file that could be located inside the project directory or inside the composer globals directory.

The content of the auth.json file is:

```json
{
    "http-basic": {
        "repo.magento.com": {
            "username": "<public-key>",
            "password": "<private-key>"
        }
    }
}
```

**Option 1: a global auth.json**

Usually the composer globals directory is `~/.composer` but you could check it by running composer `config --list --global` or `composer global config --list` looking into the home property. If the file or the directory does not exists, create it.

```
cd ~/.composer
vim auth.json
# paste auth.json template and replace username and password with valid keys
```

**Option 1: Create auth.json file for the project**

```
cd <project-dir>
vim auth.json
# paste auth.json template and replace username and password with valid keys
```

## Install project dependencies

Simply `cd` into the project directory and install the dependencies with composer:

```sh
cd <project-dir>
composer update
```

## Deploy, install magentocloud-in-docker

First move to the project directory: `cd <project-dir>`.

**Create the unified configuration file (.magento.docker.yml)**

Create the docker configuration template (aka the unified configuration file, .magento.docker.yml) inside the project directory:

```sh
vim .magento.docker.yml
```

Template:

```yml
name: magento
system:
    mode: 'production'
services:
    php:
        version: '7.3'
        extensions:
            enabled:
                - xsl
                - json
                - redis
    mysql:
        version: '10.3'
        image: 'mariadb'
    redis:
        version: '5.0'
        image: 'redis'
    elasticsearch:
        version: '7.5'
        image: 'elasticsearch'
hooks:
    build: |
        set -e
        php ./vendor/bin/ece-tools run scenario/build/generate.xml
        php ./vendor/bin/ece-tools run scenario/build/transfer.xml
    deploy: 'php ./vendor/bin/ece-tools run scenario/deploy.xml'
    post_deploy: 'php ./vendor/bin/ece-tools run scenario/post-deploy.xml'
mounts:
    var:
        path: 'var'
    app-etc:
        path: 'app/etc'
    pub-media:
        path: 'pub/media'
    pub-static:
        path: 'pub/static'
```

**Generate the docker compose configuration file (docker-compose.yml)**

Use the following command to generate a Docker Compose configuration (docker-compose.yml), the final used to build and deploy the Docker containers for your Adobe Commerce project.

```sh
./vendor/bin/ece-docker build:compose --mode="developer" --host=<vm-ip-or-vm-domain-name> --port=8080 --with-xdebug --set-docker-host
```

**Create the .env file**

docker.compose.yml config file uses the PWD environment variable to point to some directories. But when running the command with `sudo`, the PWD variable is an empty string and is not pointing to the currect directory, where we have the magento cloud project files.

This step is necessary if you need to run the docker-compose command under sudo.

```sh
echo "PWD=$PWD" > .env
```

**Destroy already existing containers first**

If the containers are already running, you need to destry them before to proceed.

TODO: I forgot the commands I've executed in this part.

Reference: https://devdocs.magento.com/cloud/docker/docker-quick-reference.html

**Start the containers**

```sh
sudo docker-compose up -d
```

**(Optional) Just build the environment**

NOTE: SKIP IT. For some reason this step fails, but fortunatelly it is not necessary for Developer mode.

Reference:
- ["Developer mode does not require the build operation."](https://devdocs.magento.com/cloud/docker/docker-mode-developer.html)
- [Build container not accessing Redis](https://github.com/magento/magento-cloud-docker/issues/275)

```sh
# Option 1: Build the application using "Magento Cloud Docker CLI"
# check: sudo ./bin/magento-docker ece-build
# Option 2: uild the environment using "Docker Compose"
sudo docker-compose run --rm build cloud-build
```

Reference: https://devdocs.magento.com/cloud/docker/docker-quick-reference.html


## (Optional) Redeploy

These commands shows a exceptional case where could be some conflicting modules in the first deployment. I this case I will be re-deploying two times just to include the modules in the required order:

```sh
# check if a required module <modulename> is enabled
sudo docker-compose run --rm deploy magento-command module:status <modulename>

# uninstall the conflicting module
composer remove <modulename2>

# DEPLOY 1: deploy, this install magento for the first time
sudo ./bin/magento-docker ece-deploy
# or sudo docker-compose run --rm deploy cloud-deploy

# install previously uninstalled package
composer require <modulename2>
composer update <modulename2>

# DEPLOY 2: deploy again to install the included module
sudo ./bin/magento-docker ece-deploy
```

## Post-deploy steps

**Create backup files and clear the cache**

```sh
sudo ./bin/magento-docker ece-post-deploy
# or sudo docker-compose run --rm deploy cloud-post-deploy
```

**Configure and connect Varnish**

```sh
sudo docker-compose run --rm deploy magento-command config:set system/full_page_cache/caching_application 2 --lock-env
sudo docker-compose run --rm deploy magento-command setup:config:set --http-cache-hosts=varnish
```

**Remove secure redirects**

```sh
sudo docker-compose run --rm deploy magento-command config:set web/secure/use_in_adminhtml 0
sudo docker-compose run --rm deploy magento-command config:set web/secure/base_url <real_sec_url>
# e.g. sudo docker-compose run --rm deploy magento-command config:set web/secure/base_url https://localhost/
TODO
```

**Disable annoying defaults**

```sh
sudo docker-compose run --rm deploy magento-command config:set admin/usage/enabled 0
sudo docker-compose run --rm deploy magento-command config:set admin/security/use_case_sensitive_login 0
sudo docker-compose run --rm deploy magento-command config:set admin/security/password_is_forced 0
sudo docker-compose run --rm deploy magento-command config:set admin/security/password_lifetime ""
sudo docker-compose run --rm deploy magento-command config:set admin/security/lockout_failures ""
# 1 year for session timeout ...
sudo docker-compose run --rm deploy magento-command config:set admin/security/session_lifetime 31536000
```

**Clean cache**

```sh
sudo docker-compose run --rm deploy magento-command cache:clean
```

## Check if magentocloud-in-docker is working

```sh
curl -v http://<host>:<port>/
# e.g. curl -v http://localhost:8080/
# e.g. curl -v http://localhost:8080/admin
```

## Get the admin credentials

```sh
cat .docker/config.php.dist
```

By defaults admin access is:

```
Username: admin
Password: 123123q
```

## Optional configure a second store

### Configure routes for separate domains

Modify the routes.yaml file: `vim .magento/routes.yaml`.

Previous:

```yml
# The routes of the project.
#
# Each route describes how an incoming URL is going to be processed.

"http://{default}/":
    type: upstream
    upstream: "mymagento:http"

"http://{all}/":
    type: upstream
    upstream: "mymagento:http"
```

Actual:

```yml
# The routes of the project.
#
# Each route describes how an incoming URL is going to be processed.

"http://{default}/":
    type: upstream
    upstream: "mymagento:http"

"http://store2.{default}/":
    type: upstream
    upstream: "mymagento:http"

"http://{all}/":
    type: upstream
    upstream: "mymagento:http"
```

### Modify Magento variables

Instead of configuring an NGINX virtual host, pass the MAGE_RUN_CODE and MAGE_RUN_TYPE variables using the magento-vars.php file located in your project root directory.

```sh
vim magento-vars.php
```

Previous
```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */

/**
 * Enable, adjust and copy this code for each store you run
 *
 * Store #0, default one
 *
 * if (isHttpHost("example.com")) {
 *    $_SERVER["MAGE_RUN_CODE"] = "default";
 *    $_SERVER["MAGE_RUN_TYPE"] = "store";
 * }
 *
 * @param string $host
 * @return bool
 */
function isHttpHost(string $host)
{
    if (!isset($_SERVER['HTTP_HOST'])) {
        return false;
    }
    return $_SERVER['HTTP_HOST'] === $host;
}
```

Actual
```php
<?php
// enable, adjust and copy this code for each store you run
// Store #0, default one
//if (isHttpHost("example.com")) {
//    $_SERVER["MAGE_RUN_CODE"] = "default";
//    $_SERVER["MAGE_RUN_TYPE"] = "store";
//}
//function isHttpHost($host)
//{
//    if (!isset($_SERVER['HTTP_HOST'])) {
//        return false;
//    }
//    return $_SERVER['HTTP_HOST'] === $host;
//}


function isHttpHost($host)
{
    if (!isset($_SERVER['HTTP_HOST'])) {
        return false;
    }
    return strpos(str_replace('---', '.', $_SERVER['HTTP_HOST']), $host) === 0;
}


if (isHttpHost("store2.")) {
    $_SERVER["MAGE_RUN_CODE"] = "store2_view";
    $_SERVER["MAGE_RUN_TYPE"] = "store";
}
```

### Create the store and store view configurations


* Go to Admin dashboard (http://localhost:8080/admin)
* Go to Stores > Settings > All Stores

Create an Store with the following information.
```
Create Store Configuration:

Web Site: Main Website
Name: Store 2
Code: store2
Root Category: Default Category
Default Store View: Store 2 View
```

Create an Store View with the following information.
```
Store: Store 2
Name: Store 2 View
Code: store2_view
Status: Enabled
Sort Order: 0
```

Applying this configuration, the table of Stores show have 2 rows. The new row should have this format:

|              |                |                     |
|--------------|----------------|---------------------|
| Main Website | Store 2        | Store 2 View        |
| (Code: base) | (Code: store2) | (Code: store2_view) |

### Change style of the two stores

This change is just to show a different color in the header section of each store, just to keep the eye aware of the store being used.

* Go to Admin dashboard (http://localhost:8080/admin)
* Go to Content > Design > Configuration

Select edit for row with "Default Store View" and append the following in the end of the content of the "HTML Head > Scripts and Style Sheets" section.

```
<style type="text/css">
.page-header {
   background-color: #d6f1ff
}
</style>
```

Select edit for row with "Store 2 View" and append the following in the end of the content of the "HTML Head > Scripts and Style Sheets" section.

```
<style type="text/css">
.page-header {
   background-color: #ffb0fe
}
</style>
```

### Add the store code to the base URL

* Go to Admin dashboard (http://localhost:8080/admin)
* Go to Stores > Settings > Configuration > General > Web
* Store View (Scope) list at the top of the page, click "Default Config"
* Expand "Url Options"
* Clear the Use system value checkbox next to "Add Store Code to Urls"
* Change "Add Store Code to Urls" to "Yes"
* Save changes and clear cache

```sh
sudo docker-compose run --rm deploy magento-command indexer:reindex
sudo docker-compose run --rm deploy magento-command cache:clean
```

### Change the default store view base URL (youcould skip this step if you see issues)

* Go to Admin dashboard (http://localhost:8080/admin)
* Go to Stores > Settings > Configuration > General > Web
* Store View (Scope) list at the top of the page, click "Default Config"
* Expand "Base URLs"
* Clear the Use system value checkbox next to "Base URLs"
* Enter the http://localhost:8080 URL in the Base URL and Base Link URL fields.
* Repeat the previous step in the Base URLs (Secure) section.
* NOTE: If you're setting up a base URL for Cloud for Adobe Commerce (in the Cloud instance), you must replace the first period with three dashes. For example, if your base URL is french.branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud, enter http://french—branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud.
* Save changes.

### Add second store domain to your hosts file

Your hosts file should have the following entries to be able to point to the two stores (default and second one).

```
localhost   localhost
localhost   store2.localhost
```

### Check second store access

Now you should be able to access to htttp://store2.localhost:8080/

## Analysis and troubleshooting

### Commands for local magentocloud-in-docker

#### Build + Deploy + Post-deploy

```sh
sudo ./bin/magento-docker ece-redeploy
```

#### Enter to bash shell for magentocloud-in-docker

```sh
sudo ./bin/magento-docker bash
# or sudo docker-compose run --rm deploy bash
```

#### Enter to the DB in database container

```sh
sudo ./bin/magento-docker ece-db
# > select * from admin_user;
```

#### Run magento commands on magentocloud-in-docker

```sh
sudo docker-compose run --rm deploy ece-command <command>
# e.g. sudo docker-compose run --rm deploy ece-command list
sudo docker-compose run --rm deploy magento-command <command>
# e.g. sudo docker-compose run --rm deploy magento-command list
```

You can run the commands of the [magento CLI](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands.html) using magento-command. The following are some examples:

```sh
# complete list of commands
sudo docker-compose run --rm deploy magento-command list

# module:status usage
sudo docker-compose run --rm deploy magento-command help module:status

# check the status if a module
sudo docker-compose run --rm deploy magento-command module:status <modulename>
```

#### Get info from magentocloud-in-docker

```sh
sudo docker-compose run --rm deploy magento-command config:show

sudo docker-compose run --rm deploy magento-command config:show web/secure/base_url
sudo docker-compose run --rm deploy magento-command config:show web/unsecure/base_url

sudo docker-compose run --rm deploy magento-command config:show:default-url
sudo docker-compose run --rm deploy magento-command config:show:store-url
# ? sudo docker-compose run --rm deploy magento-command config:show:urls
```

#### Run setup:upgrade

```sh
sudo docker-compose run --rm deploy magento-command setup:upgrade
```

#### Disable or uninstall modules

```sh
# disable
sudo docker-compose run --rm deploy magento-command module:disable <module-name>
# e.g. sudo docker-compose run --rm deploy magento-command module:disable MyModule
# e.g. sudo docker-compose run --rm deploy magento-command module:disable Mageplaza_Smtp
# e.g. sudo docker-compose run --rm deploy magento-command module:disable Mageplaza_Core
sudo docker-compose run --rm deploy magento-command cache:clean

# uninstall
sudo docker-compose run --rm deploy magento-command module:uninstall <module-name>
# e.g. sudo docker-compose run --rm deploy magento-command module:uninstall Mageplaza_Smtp
# e.g. sudo docker-compose run --rm deploy magento-command module:uninstall Mageplaza_Core
```

#### Useful magentocloud-in-docker management commands

```sh
sudo docker-compose run --rm deploy magento-command setup:upgrade

sudo docker-compose run --rm deploy magento-command indexer:info
sudo docker-compose run --rm deploy magento-command indexer:status
sudo docker-compose run --rm deploy magento-command indexer:reindex

sudo docker-compose run --rm deploy magento-command setup:di:compile
sudo docker-compose run --rm deploy magento-command setup:static-content:deploy

sudo docker-compose run --rm deploy magento-command cache:status
sudo docker-compose run --rm deploy magento-command cache:clean
sudo docker-compose run --rm deploy magento-command cache:flush
```

#### Useful magentocloud-in-cloud management commands

```sh
cd <project-dir>
magento-cloud ssh
php bin/magento indexer:reindex
```

#### Start, Stop and Restart

```sh
sudo ./bin/magento-docker start
sudo ./bin/magento-docker stop
sudo ./bin/magento-docker restart
```

#### Destroy all containers and volumes

```sh
sudo ./bin/magento-docker down
# or sudo docker-compose down -v
```

### Troubleshooting

#### List listening ports

```sh
sudo netstat -tulpn | grep LISTEN
```

#### Start an HTTP server for testing

```sh
docker run -p 8000:8000 -it python:3.7-slim python3 -m http.server --bind 0.0.0.0
```

#### Bash into a container

```sh
sudo docker exec -it <container-name> bash
# e.g. sudo docker exec -it project_web_1 bash
```

#### Get container details

```sh
sudo docker inspect <container-name>
# e.g. sudo docker inspect project_web_1
```

#### Iptables issue when trying to restart containers

If you see any IPTABLES related issue when trying to restart the containers through docker-compose, then try the following commands.

```sh
systemctl stop firewalld
# systemctl disable firewalld # if needed
sudo iptables -t filter -F
sudo iptables -t filter -X
sudo systemctl restart docker
sudo docker-compose restart
```

#### Connection reset when trying to connect from host to container

Test:

```sh
# terminal 1
docker run -p 8000:8000 -it python:3.7-slim python3 -m http.server --bind 0.0.0.0
# terminal 2
curl -v localhost:8000
```

Solution:

```sh
sudo systemctl stop docker
sudo ip link set dev docker0 down
sudo ip link delete docker0
sudo ip link add name docker0 type bridge
sudo ip addr add 172.16.10.1/24 dev docker0
sudo ip link set dev docker0 up
sudo systemctl restart docker
```

Reference: https://www.programmersought.com/article/34376875862/

## Usage: Magento Cloud CLI

### Getting help

```sh
# list available commands
magento-cloud list

# getting help of an specific command, for example "get"
magento-cloud get --help
```

### Show projects and environments information

```sh
# list projects
magento-cloud projects
# e.g.
# my project (ID: jklhc54pyvnxi)

# show project info
magento-cloud project:info -p <project-id>

# show environments info
magento-cloud environments -p <project-id>
#  e.g.
# | ID               | Title        | Status |
# | master           | Master       | Active |
# |   staging        | staging      | Active |
# |     my-own-env   | my-own-env   | Active |
```

### Show information from project context

The following commands shows how to get inforomation being inside the project directory.

First move to the project directory: `cd <project-dir>`.

```sh
# show environments
# option 1, in the output, the * mark says the current environment
magento-cloud environments
# option 2
git branch --remote

# move to another environment
magento-cloud checkout <env-id>
```

**Changing environment (branch)**

```sh
# move to another environment
magento-cloud checkout <env-id>
```

**List non-commited changes in environment (branch)**

```sh
git status
```

## Usage: ece-tools and ece-docker commands

The following commands shows how to use "ece-tools" and "ece-docker", dependencies listed in the project.

First move to the project directory: `cd <project-dir>`.

### Getting help

```sh
./vendor/bin/ece-tools list
./vendor/bin/ece-tools cloud:config:validate --help
./vendor/bin/ece-docker list
./vendor/bin/ece-docker build:compose --help
```

## Reference links

- Magento Cloud official guide - [https://devdocs.magento.com/cloud/docker/docker-development.html](https://devdocs.magento.com/cloud/docker/docker-development.html).
- magento-cloud repo - These are the base files used for creating a magento cloud project.
- magento-cloud-docker - Package used for managing docker environment with magento. This repo contains the definition files for docker containers created in local environment.
- [Development Environment setup using Magento Cloud Docker](https://gist.github.com/jayachandraoggy/c7539b91c10d28abfa877c67d78af459)
- [How To Communicate Between Docker Containers](https://www.tutorialworks.com/container-networking/#:~:text=Check that the bridge network,bridge network in the list.&text=Start your containers%3A Start your,it to the bridge network.)
- [How to install and enable EPEL repository on a CentOS/RHEL 7](https://www.cyberciti.biz/faq/installing-rhel-epel-repo-on-centos-redhat-7-x/)
