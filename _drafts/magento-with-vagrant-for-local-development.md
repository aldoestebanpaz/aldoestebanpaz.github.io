# Magento with Vagrant for local development

## Installation

Open a administrative bash shell and follow the steps bellow.

NOTE: I'll be using the bash shell provided by my installation of git on Windows.

### Install vagrant

1. Download the new package from `https://www.vagrantup.com/downloads`.
2. Install it (even an old installation because it is like updating it).
3. Update the provider VirtualBox: `vagrant box update`.
4. Install the required plugins for vagrant:
```sh
# VirtualBox Guest Additions up to date (winnfsd requires vbguest 0.29.0)
vagrant plugin install vagrant-vbguest --plugin-version=0.29.0
# NFS  support for Windows
vagrant plugin install vagrant-winnfsd
# Manages the 'hosts' file on guest machines (and optionally the host).
vagrant plugin install vagrant-hostmanager
```

### Copy the files for VM creation and provisioning

```sh
cd $USERPROFILE
mkdir -p ./vms/magento
cp $PROJECT/{Vagrantfile.template,provision.sh} ./vms/magento
cd ./vms/magento
mkdir magentoee
mkdir magentocloud
mv Vagrantfile.template Vagrantfile
# Replace values below
sudo sed -i 's/{{USER_NAME}}/aldo.paz/i' Vagrantfile
sudo sed -i 's/{{USER_EMAIL}}/aldo.paz@noemail.com/i' Vagrantfile
sudo sed -i 's/{{MAGENTO_REPO_PUBLIC_KEY}}/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/i' Vagrantfile
sudo sed -i 's/{{MAGENTO_REPO_PRIVATE_KEY}}/yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy/i' Vagrantfile
sudo sed -i 's/{{CLOUD_API_TOKEN}}/zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz/i' Vagrantfile
```

### Initialize the VM

Create a new VM with vagrant, start it, and start an interactive shell (ssh) to configure it:

```sh
vagrant up
vagrant hostmanager
vagrant ssh
```

### Install and configure magentoee

1. Go to the synced directory and create a new project:

```sh
su - magento
MAGENTOEE_VERSION="2.3.4"
cd /usr/share/magentoee
COMPOSER_MEMORY_LIMIT=3G composer create-project --repository-url=https://repo.magento.com/ magento/project-enterprise-edition=$MAGENTOEE_VERSION .
```

2. Install it:

```sh
cd /usr/share/magentoee/bin

php magento setup:install \
  --base-url=http://magento.local/ \
  --db-host=localhost \
  --db-name=magentoee \
  --db-user=magento \
  --db-password=magento \
  --admin-firstname=admin \
  --admin-lastname=admin \
  --admin-email=admin@admin.com \
  --admin-user=admin \
  --admin-password=admin123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/Chicago \
  --use-rewrites=1 \
  --backend-frontname=admin \
  --admin-use-security-key=0 \
  --session-save=files \
  --cleanup-database

php magento setup:upgrade
php magento setup:di:compile
php magento setup:static:deploy -f

cd ..
find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +

chown -R :nginx .
chmod u+x bin/magento

exit
```

3. Configure nginx

```sh
sudo tee /etc/nginx/conf.d/magento.conf > /dev/null <<EOT
upstream fastcgi_backend {
  server  unix:/run/php73-fpm.sock;
}

server {
  listen 80;
  server_name magento.local;
  set \$MAGE_ROOT /usr/share/magentoee;
  include /usr/share/magentoee/nginx.conf.sample;
}
EOT

sudo nginx -t
sudo service nginx restart
```

### Install and configure magentocloud

1. Search the project to clone:

```sh
# list project IDs
magento-cloud project:list
# clone the project
magento-cloud project:get <ID>
```

2. Go to the synced directory and download the project:

```sh
su - magento
CLOUD_PROJECT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxx
cd /usr/share/magentocloud
magento-cloud project:get -e staging $CLOUD_PROJECT_ID mycloudproject --yes
cd ./mycloudproject
composer install
```

3. Go to the magento directory and install it:

```sh
cd /usr/share/magentocloud/mycloudproject/bin

php magento setup:install \
  --base-url=http://magento.local/ \
  --db-host=localhost \
  --db-name=magentocloud \
  --db-user=magento \
  --db-password=magento \
  --admin-firstname=admin \
  --admin-lastname=admin \
  --admin-email=admin@admin.com \
  --admin-user=admin \
  --admin-password=admin123 \
  --language=en_US \
  --currency=USD \
  --timezone=America/Chicago \
  --use-rewrites=1 \
  --backend-frontname=admin \
  --admin-use-security-key=0 \
  --session-save=files \
  --cleanup-database

php magento setup:upgrade
php magento setup:di:compile
php magento setup:static:deploy -f

cd ..
find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +

chown -R :nginx .
chmod u+x bin/magento

exit
```

4. Configure nginx

```sh
sudo tee /etc/nginx/conf.d/magento.conf > /dev/null <<EOT
upstream fastcgi_backend {
  server  unix:/run/php73-fpm.sock;
}

server {
  listen 80;
  server_name magento.local;
  set \$MAGE_ROOT /usr/share/magentocloud/mycloudproject;
  include /usr/share/magentocloud/mycloudproject/nginx.conf.sample;
}
EOT

sudo nginx -t
sudo service nginx restart
```

### Enable remote debugging

1. Open and edit xdebug.ini:

```sh
sudo vi /etc/opt/remi/php73/php.d/15-xdebug.ini
```

2. Add the following definitions:

```ini
xdebug.remote_connect_back = true
xdebug.remote_enable = true
```

3. Restart php-fpm

```sh
sudo service php73-php-fpm restart
```

4. Open the shared folder in host with vscode and create this debug configuration:

```sh
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000,
            "pathMappings": {
                "/usr/share/magentoee": "${workspaceRoot}",
            },
            "xdebugSettings": {
                "max_children": 2000,
                "max_data": -1

            }
        }
    ]
}
```

Replace `/usr/share/magentoee` to `/usr/share/magentocloud` when using the project for cloud.

## (Optional) configure a second store and a second website

### Configure hostmanager aliases

Add aliases on your `Vagrantfile`:

```ruby
config.hostmanager.aliases = %w(store2.magento.local website2.magento.local)
```

And run `vagrant hostmanager` if you did any change.

### Configure routes for separate domains and modify Magento variables

We have to configure a Nginx virtual hosts and set the MAGE_RUN_CODE and MAGE_RUN_TYPE variables.

In the following example I use `/usr/share/magentoee`, change paths to `/usr/share/magentocloud/mycloudproject` if you are configuring the non-cloud project:

```sh
sudo tee /etc/nginx/conf.d/magento.conf > /dev/null <<EOT
upstream fastcgi_backend {
  server  unix:/run/php73-fpm.sock;
}

# Sets \$MAGE_RUN_CODE either '', store2_view or base2 depending on the value of \$http_host.
map \$http_host \$MAGE_RUN_CODE {
    default '';
    store2.magento.local store2_view;
    website2.magento.local base2;
}

# Sets \$MAGE_RUN_TYPE either '', store or website depending on the value of \$http_host.
map \$http_host \$MAGE_RUN_TYPE {
    default '';
    store2.magento.local store;
    website2.magento.local website;
}

server {
  listen 80;
  server_name magento.local store2.magento.local website2.magento.local;
  set \$MAGE_ROOT /usr/share/magentoee;
  # set \$MAGE_MODE developer;
  include /usr/share/magentoee/nginx.conf;
}
EOT
```

Pass the variables into the "PHP entry point for main application" block in the included .conf file:

```sh
cd /usr/share/magentoee/mycloudproject
cp nginx.conf.sample nginx.conf
# This command inserts the lines...
#   fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
#   fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;
# after the line  ...
# location ~ ^/(index|get|static|errors/report|errors/404|errors/503|health_check)\.php$ {
# the "PHP entry point for main application" block.
sed -i '/^location \~.*\\\.php\$ {/a\
    fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;\
    fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;\
' nginx.conf
```

Finally restart nginx:

```sh
sudo nginx -t
sudo service nginx restart
```

### Configurations for Store2

#### Create the store and store view configurations for "store2"

- Go to Admin dashboard (http://magento.local/admin)
- Go to Stores > Settings > All Stores

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

#### Change style of the two stores

This change is just to show a different color in the header section of each store, just to keep the eye aware of the store being used.

* Go to Admin dashboard (http://magento.local/admin)
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

#### Add the store code to the base URL

Magento gives you the option to add the store code to the site base URL, which simplifies the process of setting up multiple stores. Using this option, you do not have to create directories on the Magento file system to store index.php and .htaccess.

This prevents index.php and .htaccess from getting out of sync with the Magento codebase in future upgrades.

* Go to Admin dashboard (http://magento.local/admin)
* Go to Stores > Settings > Configuration > General > Web
* Switch the Scope (at the top of the page) to "Default Config"
* Expand "Url Options"
* Clear the Use system value checkbox next to "Add Store Code to Urls"
* Change "Add Store Code to Urls" to "Yes"
* Save changes and clear cache

```sh
sudo docker-compose run --rm deploy magento-command indexer:reindex
sudo docker-compose run --rm deploy magento-command cache:clean
```

#### Change the base URL of the default store

* Go to Admin dashboard (http://magento.local/admin)
* Go to Stores > Settings > Configuration > General > Web
* Switch the Scope (at the top of the page) to "Default Config"
* Expand "Base URLs"
* Clear the Use system value checkbox next to "Base URLs"
* Enter the http://magento.local URL in the Base URL and Base Link URL fields.
* Repeat the previous step in the Base URLs (Secure) section.
* Save changes.

#### Change the base URL for "Store 2 View"

* Go to Admin dashboard (http://magento.local/admin)
* Go to Stores > Settings > Configuration > General > Web
* Switch the Scope (at the top of the page) to "Store 2 View"
* Expand "Base URLs"
* Clear the Use system value checkbox next to "Base URL"
* Enter the http://store2.magento.local/ URL in the Base URL and Base Link URL fields.
* Repeat the previous step in the Base URLs (Secure) section, but replace http by https.
* NOTE: I didn't needed it, but you could check if it works. If you're setting up a base URL for Cloud for Adobe Commerce (in the Cloud instance), you must replace the first period with three dashes. For example, if your base URL is french.branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud, enter http://french---branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud.
* Save changes.

#### Add second store subdomain to your hosts file

Your hosts file should have the following entries to be able to point to the two stores (default and second one).

```
localhost   localhost
localhost   store2.localhost
```

#### Check access to the second store

Now you should be able to access to htttp://store2.magento.local/

### Configurations for website2

#### Create the websiten store and store view configurations for "website2"

- Go to Admin dashboard (http://magento.local/admin)
- Go to Stores > Settings > All Stores

Create a Website with the following information.
```
Name: Website2
Code: base2
Sort Order: 0
Default Store: Website2 Store
Set as default: false
```

Create a Store with the following information.
```
Web Site: Website2
Name: Website2 Store
Code: website2_store
Root Category: Default Category
Default Store View: Website2 Default Store View
```

Create a Store View with the following information.
```
Store: Website2 Store
Name: Website2 Default Store View
Code: website2_default
Status: Enabled
Sort Order: 0
```

Applying this configuration, the table of Stores show have 2 rows. The new row should have this format:

|               |                        |                             |
|---------------|------------------------|-----------------------------|
| Website2      | Website2 Store         | Website2 Default Store View |
| (Code: base2) | (Code: website2_store) | (Code: website2_default)    |

#### Change the base URL for Website2

* Go to Admin dashboard (http://magento.local/admin)
* Go to Stores > Settings > Configuration > General > Web
* Switch the Scope (at the top of the page) to "Website2"
* Expand "Base URLs"
* Clear the Use system value checkbox next to "Base URL"
* Enter the http://website2.magento.local/ URL in the Base URL and Base Link URL fields.
* Repeat the previous step in the Base URLs (Secure) section, but replace http by https.
* NOTE: I didn't needed it, but you could check if it works. If you're setting up a base URL for Cloud for Adobe Commerce (in the Cloud instance), you must replace the first period with three dashes. For example, if your base URL is french.branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud, enter http://french---branch-xXxXxXx-xXxXxXxXxXxXx.us.magentosite.cloud.
* Save changes.

#### Add website2 subdomain to your hosts file

Your hosts file should have the following entries to be able to point to the two stores (default and second one).

```
localhost   localhost
localhost   website2.localhost
```

#### Check access to the second website

Now you should be able to access to htttp://website2.magento.local/


## (Optional) configure a second store and a second website in Magento Cloud

These steps are needed for creating the store2 and website2 domains in Magento Cloud, not for the current development VM.

### Configure routes for separate domains

Modify the routes.yaml file: `vim .magento/routes.yaml`.

Before:

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

After:

```yml
# The routes of the project.
#
# Each route describes how an incoming URL is going to be processed.

"http://{default}/":
    type: upstream
    upstream: "mymagento:http"

# for testing a different store, and his own store view, in the same Main Website.
"http://store2.{default}/":
    type: upstream
    upstream: "mymagento:http"

# for testing a different website, with his own store and store view.
"http://website2.{default}/":
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

Before:

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

After:

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
    // If MAGE_RUN_TYPE" == "store" then MAGE_RUN_CODE must point to the code of a store view.
    $_SERVER["MAGE_RUN_CODE"] = "store2_view";
    $_SERVER["MAGE_RUN_TYPE"] = "store";
} elseif (isHttpHost("website2.")) {
    // If MAGE_RUN_TYPE" == "website" then MAGE_RUN_CODE must point to the code of a website.
    $_SERVER["MAGE_RUN_CODE"] = "base2";
    $_SERVER["MAGE_RUN_TYPE"] = "website";
}
```

### Next steps

Follow the steps explained in the previous section about configuring store2 and website2.

## Troubleshooting

### Errors with vagrant

* For `VERR_INTNET_FLT_IF_NOT_FOUND` on Windows 10, look at: https://stackoverflow.com/questions/33725779/failed-to-open-create-the-internal-network-vagrant-on-windows10.
* For 'No package kernel-devel-3.10.0-1127.el7.x86_64 available', look at: https://github.com/dotless-de/vagrant-vbguest/issues/399.
* For 'How to uninstall the multiple Host-Only interfaces created on Windows?', look at: https://stackoverflow.com/questions/32093792/how-to-remove-extra-host-only-network-interfaces-created-by-vagrant-on-windows-1.
* If you want to delete the VM and create a new one, then run the following commands:
```sh
vagrant destroy
vagrant up
```

### Updates in vagrant

You can make changes in the `Vagrantfile` and apply the updates to the running VM with the following command:

```sh
vagrant reload
```

### CentOS

**Verify EPEL is enabled**
```sh
yum repolist epel
# or
yum repolist all
```

**Verify REMI is enabled**
```sh
yum repolist remi
# or
yum repolist all
```

**firewalld**

Check firewalld status:

```sh
service firewalld status
```

If firewalld is enabled, then allow HTTP and HTTPS traffic:

```sh
sudo firewall-cmd --permanent --zone=public --add-service=http ;
sudo firewall-cmd --permanent --zone=public --add-service=https ;
sudo firewall-cmd --reload
```

### Nginx and FPM

**server user and group**

```sh
# Find the user of the running process (in this case the user is nginx)
ps aux | grep nginx
# Find the group of that user (in this case the group is nginx)
groups nginx
```

You can check the status running the following commands:

```sh
sudo systemctl status php73-php-fpm
sudo systemctl status nginx
# or
service nginx status
service php73-php-fpm status
```

Check if FPM is running:

```sh
netstat -pl | grep php73-fpm.sock
```

See the logs in nginx with one of the following commands:

```sh
sudo journalctl -xe
# or
sudo tail -f /var/log/nginx/error.log
# or
sudo tail -f /var/log/nginx/access.log
```

And restart them with:

```sh
sudo systemctl restart php73-php-fpm
sudo systemctl restart nginx
# or
sudo service nginx restart
sudo service php73-php-fpm restart
```

### NFS service and access

```sh
# Enable the NFS service
sudo systemctl enable nfs
# Enable serving files from NFS mounts:
sudo setsebool -P httpd_use_nfs 1
```

### PHP

**Get location of php.ini**

```sh
php -i | grep php.ini
```

**List locations of all the .ini files**

```sh
php -i | grep .ini
```

**List installed modules**

```sh
php -m
```

### Magento

**List all config values in Magento**

```sh
./bin/magento config:show
```

**Check Admin URL**

```
php magento info:adminuri
```

**Set a config value**

You can do it with commands. Example:

```sh
cd $MAGENTO_DIR/bin
php magento config:set web/unsecure/base_url http://192.168.33.10/
php magento config:set web/secure/base_url http://192.168.33.10/
php magento cache:flush
```

Or using SQL:

```sh
mysql -u magento -p magento
select * from core_config_data where path like '%base%url%';
update core_config_data set value = 'http://192.168.33.10/' where path = 'web/unsecure/base_url';
update core_config_data set value = 'https://192.168.33.10/' where path = 'web/secure/base_url';
exit
cd $MAGENTO_DIR/bin
php magento cache:flush
```

**Flush cache**

```sh
cd $MAGENTO_DIR/bin
php magento cache:flush
```

### Magento Cloud CLI

**List available commands**

```sh
magento-cloud list
```

**List environment variables in Cloud instance**

```sh
cd $MAGENTOCLOUD_DIR
magento-cloud variable:get
```

## Project files

**Vagrantfile.template**

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
    # Customize the amount of memory on the VM:
    vb.memory = 4096
    vb.cpus = 2
  end

  # Required for centos/7 or centos/8
  # Alternative 1: Install the latest kernel and perform a reboot of the Vagrant VM before continuing the provisioning.
  config.vbguest.installer_options = { allow_kernel_upgrade: true }
  # Alternative 2: It won't raise error on provision and won't install Virtualbox Guest Additions as well.
  # Config.vbguest.auto_update = false

  config.vm.hostname = "magento.local"

  config.hostmanager.aliases = %w(store2.magento.local website2.magento.local)
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.include_offline = true

  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.synced_folder "./magentoee", "/usr/share/magentoee",
    type:"nfs",
    create: true,
    # group:'nginx', owner:'magento', # no supported by NFS
    mount_options: %w{rw,async,fsc,nolock,vers=3,udp,rsize=32768,wsize=32768,hard,noatime,actimeo=2}
  config.vm.synced_folder "./magentocloud", "/usr/share/magentocloud",
    type:"nfs",
    create: true,
    # group:'nginx', owner:'magento', # no supported by NFS
    mount_options: %w{rw,async,fsc,nolock,vers=3,udp,rsize=32768,wsize=32768,hard,noatime,actimeo=2}

  config.vm.provision "shell" do |s|
    s.path = "provision.sh"
    s.args   = [
      "{{USER_NAME}}",
      "{{USER_EMAIL}}",
      "{{MAGENTO_REPO_PUBLIC_KEY}}",
      "{{MAGENTO_REPO_PRIVATE_KEY}}",
      "{{CLOUD_API_TOKEN}}"
    ]
  end
end

```

**provision.sh**

```sh
set -x #echo on

USER_NAME=$1
USER_EMAIL=$2
MAGENTO_REPO_PUBLIC_KEY=$3
MAGENTO_REPO_PRIVATE_KEY=$4
CLOUD_API_TOKEN=$5

sudo yum -y update

sudo yum install curl
sudo yum install ca-certificates
sudo update-ca-trust enable
sudo update-ca-trust extract

sudo yum -y install epel-release
# or
# sudo yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi

sudo yum -y update

sudo yum -y install yum-utils net-tools lsof unzip git

sudo yum -y install nginx
sudo setsebool httpd_use_nfs 1
sudo chown -R nginx:nginx /usr/share/nginx/html
sudo chmod -R g+w /usr/share/nginx/html
sudo systemctl enable nginx
sudo systemctl start nginx

sudo systemctl stop firewalld
sudo systemctl disable firewalld

sudo yum -y install php73 php73-php-fpm
sudo ln -s /usr/bin/php73 /usr/bin/php
sudo yum -y install \
    php73-php-pecl-xdebug \
    php73-php-json \
    php73-php-bcmath \
    php73-php-opcache \
    php73-php-common \
    php73-php-intl \
    php73-php-xml \
    php73-php-gd \
    php73-php-mbstring \
    php73-php-pdo \
    php73-php-soap \
    php73-php-mysqlnd \
    php73-php-pecl-zip \
    php73-php-process

curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
sudo composer self-update --1
composer config --global process-timeout 300

sudo sed -r -i.bak 's/^;?cgi\.fix_pathinfo\s*=.*/cgi.fix_pathinfo = 0/i' /etc/opt/remi/php73/php.ini && \
    sudo sed -r -i.bak 's/^;?memory_limit\s*=.*/memory_limit = 2G/i' /etc/opt/remi/php73/php.ini && \
    sudo sed -r -i.bak 's/^;?max_execution_time\s*=.*/max_execution_time = 1800/i' /etc/opt/remi/php73/php.ini && \
    sudo sed -r -i.bak 's/^;?zlib\.output_compression\s*=.*/zlib.output_compression = On/i' /etc/opt/remi/php73/php.ini && \
    sudo sed -r -i.bak 's/^;?session\.save_path\s*=.*/session.save_path = "\/var\/opt\/remi\/php73\/lib\/php\/session"/i' /etc/opt/remi/php73/php.ini
sudo mkdir -p /var/opt/remi/php73/lib/php/session
sudo chown -R nginx:nginx /var/opt/remi/php73
sudo sed -r -i.bak 's/^user\s*=.*/user = nginx/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^group\s*=.*/group = nginx/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^listen\s*=.*/listen = \/run\/php73-fpm.sock/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?listen.owner\s*=.*/listen.owner = nginx/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?listen.group\s*=.*/listen.group = nginx/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?listen.mode\s*=.*/listen.mode = 0660/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?(env\[HOSTNAME\]\s*=.*)/\1/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?(env\[PATH\]\s*=.*)/\1/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?(env\[TMP\]\s*=.*)/\1/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?(env\[TMPDIR\]\s*=.*)/\1/i' /etc/opt/remi/php73/php-fpm.d/www.conf && \
    sudo sed -r -i.bak 's/^;?(env\[TEMP\]\s*=.*)/\1/i' /etc/opt/remi/php73/php-fpm.d/www.conf
sudo mkdir -p /run/php73-fpm
sudo chown -R nginx:nginx /run/php73-fpm
sudo systemctl enable php73-php-fpm
sudo systemctl start php73-php-fpm

curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
sudo yum clean all
sudo yum -y install MariaDB-server MariaDB-client
sudo systemctl enable mariadb
sudo systemctl start mariadb

# Extracted from script 'mysql_secure_installation' or 'mariadb-secure-installation'
sudo mariadb -u root -e "DELETE FROM mysql.global_priv WHERE User='';"
sudo mariadb -u root -e "DROP DATABASE IF EXISTS test;"
sudo mariadb -u root -e "DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%'"
sudo mariadb -u root -e "UPDATE mysql.global_priv SET priv=json_set(priv, '$.plugin', 'mysql_native_password', '$.authentication_string', PASSWORD('root')) WHERE User='root';"
sudo mariadb -u root -e "FLUSH PRIVILEGES;"

sudo mariadb -u root -e "CREATE DATABASE magento;"
sudo mariadb -u root -e "GRANT ALL PRIVILEGES ON magentoee.* TO 'magento'@'localhost' IDENTIFIED BY 'magento';"
sudo mariadb -u root -e "CREATE DATABASE magentocloud;"
sudo mariadb -u root -e "GRANT ALL PRIVILEGES ON magentocloud.* TO 'magento'@'localhost' IDENTIFIED BY 'magento';"

sudo adduser magento
echo "magento" | sudo passwd magento  --stdin
sudo usermod -a -G nginx magento
echo magento | su - magento -c "\
    curl -sS https://accounts.magento.cloud/cli/installer | php && \
    echo \"export MAGENTO_CLOUD_CLI_TOKEN=$CLOUD_API_TOKEN\" >> ~/.bashrc && \
    source ~/.bashrc && \
    mkdir ~/.composer && \
    echo -e \"{ \\\"http-basic\\\": { \\\"repo.magento.com\\\": { \\\"username\\\": \\\"$MAGENTO_REPO_PUBLIC_KEY\\\", \\\"password\\\": \\\"$MAGENTO_REPO_PRIVATE_KEY\\\" } } }\" > ~/.composer/auth.json && \
    composer config --global process-timeout 300 && \
    composer clear-cache && \
    git config --global user.email "$USER_EMAIL" && \
    git config --global user.name "$USER_NAME" && \
    ssh-keygen -t rsa -b 4096 -N \"\" -C \"$USER_EMAIL\" -f ~/.ssh/id_rsa_magento && \
    echo -e \"Host *\\n    AddKeysToAgent yes\\n    IdentityFile ~/.ssh/id_rsa_magento\" >> ~/.ssh/config && \
    echo -e \"Host *.magento.cloud\\n    StrictHostKeyChecking no\" >> ~/.ssh/config && \
    chmod 600 ~/.ssh/config && \
    eval \"\$(ssh-agent -s)\" && \
    ssh-add ~/.ssh/id_rsa_magento && \
    magento-cloud ssh-key:add ~/.ssh/id_rsa_magento.pub --yes
"

```
