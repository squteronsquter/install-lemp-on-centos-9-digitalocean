# Configure LEMP on Centos 9 Digital Ocean droplet.

## After you have created your droplet and have the IP <ip-address-of-your-server>

```
ssh root@<ip-address-of-your-server> -i ~/.ssh/<your_ssh_key_name>
```
## Change root password

```
passwd
```

## Make note for yourself so that you do not lose or forget your password
u: root
p: <your-secret-password>

# Initial Centos setup: <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-8>

```
adduser <your-new-not-root-user-name>

passwd <your-new-not-root-user-name>

```

## Remember or write down

p: <your-new-generated-strong-password>

## Make the new user a sudoer (with root rights)

```
usermod -aG wheel <your-new-not-root-user-name>
```

## Install and configure firewall

```
dnf install firewalld -y

systemctl start firewalld

firewall-cmd --permanent --list-all

firewall-cmd --get-services

firewall-cmd --permanent --add-service=http

firewall-cmd --reload

rsync --archive --chown=<your-new-not-root-user-name>:<your-new-not-root-user-name> ~/.ssh /home/<your-new-not-root-user-name>
```

## add ssh keys for new user // as root in the root directory run

```
cp -r .ssh/ /home/<your-new-not-root-user-name>/

ls -laht /home/<your-new-not-root-user-name>/

ls -laht /home/<your-new-not-root-user-name>/.ssh  
```


## Authorized keys file should have 600 permissions - rerun chown to make .ssh folder and files owned by <your-new-not-root-user-name>

```
rsync --archive --chown=<your-new-not-root-user-name>:<your-new-not-root-user-name> ~/.ssh /home/<your-new-not-root-user-name>
```

## Login remotely as your new user with the ssh keys and check if he can sudo

```
ssh <your-new-not-root-user-name>@<your-droplet-ip-address> -i ~/.ssh/digitalocean_thinkinink

sudo yum update

```

## LEMP: <https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-8>

## Install and configure Nginx

```
sudo dnf install nginx

sudo systemctl start nginx

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --list-all

sudo firewall-cmd --reload
```

## Install and configure MariaDB with the secure installation option

```
sudo dnf install mariadb-server

sudo systemctl start mariadb

sudo mysql_secure_installation

sudo mysql
```

## Sample commands to create databases and users

```
CREATE DATABASE example_database;

GRANT ALL ON example_database.* TO 'example_user'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;

FLUSH PRIVILEGES;

exit;

mysql -u example_user -p
```

# Install PHP FPM - if your will ever need it

```
sudo dnf install php-fpm php-mysqlnd

sudo dnf install vim

sudo vim /etc/php-fpm.d/www.conf

```

## Edit config files

Now look for the user and group directives

If you are using nano, you can hit CTRL+W to search for these terms inside the open file.

If you are using vim, you can "/" to search and find strings

### /etc/php-fpm.d/www.conf

```
 ; Unix user/group of processes
 ; Note: The user is mandatory. If the group is not set, the default user's group will be used
 ; RPM: apache user chosen to provide access to the same directories as httpd
 user = apache
 ; RPM: Keep a group allowed to write in log dir.
 group = apache

```

## You’ll notice that both the user and group variables are set to apache. We need to change these to nginx

### /etc/php-fpm.d/www.conf

```
; RPM: apache user chosen to provide access to the same directories as httpd
user = nginx
; RPM: Keep a group allowed to write in log dir.
group = nginx

```

## Restart PHP FPM

```
sudo systemctl start php-fpm

sudo systemctl restart nginx

```

## Test php

```
sudo chown -R <your-new-not-root-user-name>.<your-new-not-root-user-name> /usr/share/nginx/html/

vim /usr/share/nginx/html/info.php

<?php

phpinfo();

```

## Check if works!

```
(http://<your-droplet-ip-address>/info.php)
```

### Mine at the time of this note it reads: PHP Version 8.0.13

## Default nginx webserver responds at

```
(http://<your-droplet-ip-address>/)
```

## Clean the default nginx root folder

```
cd /usr/share/nginx/html/

rm -y info.php

```

## Configure domains:

<YOUR-REGISTERED-DOMAIN.TLD>

<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>

## Create directories for the new domains

```
cd /usr/share/nginx # or user full paths

mkdir -p /usr/share/nginx/<YOUR-REGISTERED-DOMAIN.TLD>/public_html

mkdir -p /usr/share/nginx/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/public_html

vim /usr/share/nginx/<YOUR-REGISTERED-DOMAIN.TLD>/public_html/index.html

vim /usr/share/nginx/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/public_html/index.html
```

## Change owner of the nginx root folder to: <your-new-not-root-user-name>

```

chown <your-new-not-root-user-name>:<your-new-not-root-user-name> /usr/share/nginx/

chown <your-new-not-root-user-name>:<your-new-not-root-user-name> /usr/share/nginx/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/

chown <your-new-not-root-user-name>:<your-new-not-root-user-name> /usr/share/nginx/<YOUR-REGISTERED-DOMAIN.TLD>/

sudo chmod -R 755 /var/www
```


## Check if the folders have been created

```
ls -laht /usr/share/nginx/
```


## Configure Nginx to use theses domains:

### https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-centos-7

```
sudo mkdir /etc/nginx/sites-available

sudo mkdir /etc/nginx/sites-enabled
```

## Tell Nginx to use these directories

```
sudo vim /etc/nginx/nginx.conf
```


## Edit the file above by adding these line to the end of the http {} block:

```
include /etc/nginx/sites-enabled/*.conf;

server_names_hash_bucket_size 64;
```

## Check configuration and reload/start/stop nginx

```
sudo nginx -t

sudo systemctl status nginx

sudo systemctl start nginx

sudo systemctl reload nginx
```


## Enable Nginx to start automatically after system restart

```
systemctl enable nginx
```


## Create config file for each domain

```
sudo cp /etc/nginx/conf.d/default.conf /etc/nginx/sites-available/<YOUR-REGISTERED-DOMAIN.TLD>.conf

sudo cp /etc/nginx/conf.d/default.conf /etc/nginx/sites-available/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>.conf
```

## Config file for <YOUR-REGISTERED-DOMAIN.TLD> should look like this:

```
server {
    listen  80;

    server_name <YOUR-REGISTERED-DOMAIN.TLD> www.<YOUR-REGISTERED-DOMAIN.TLD>;

    location / {
        root  /usr/share/nginx/<YOUR-REGISTERED-DOMAIN.TLD>/public_html;
        index  index.html index.htm;
        try_files $uri $uri/ =404;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/public_html;
    }
}
```

## Config file for <SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD> should look like this:

```
server {
    listen  80;

    server_name <SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD> www.<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>;

    location / {
        root  /usr/share/nginx/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/public_html;
        index  index.html index.htm;
        try_files $uri $uri/ =404;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/public_html;
    }
}
```

## Create symbolic links to sites-enabled

```
sudo ln -s /etc/nginx/sites-available/<YOUR-REGISTERED-DOMAIN.TLD>.conf /etc/nginx/sites-enabled/<YOUR-REGISTERED-DOMAIN.TLD>.conf

sudo ln -s /etc/nginx/sites-available/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>.conf /etc/nginx/sites-enabled/<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>.conf
```

## Check configuration and reload nginx

```
sudo nginx -t

sudo systemctl reload nginx
```

## Visit the new website

(http://<YOUR-REGISTERED-DOMAIN.TLD>/)

(http://<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/)

## Let's Encrypt installation with Certbot

## Check if snaptd is installed (should be preinstalled on Centos)

## If not preinstalled install like that:

```
sudo dnf install epel-release

sudo dnf upgrade

sudo yum install snapd

sudo systemctl enable --now snapd.socket

sudo ln -s /var/lib/snapd/snap /snap

sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot

sudo certbot --nginx

sudo certbot renew --dry-run

```

## Confirm

(https://<YOUR-REGISTERED-DOMAIN.TLD>)

(https://<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>)


## If there is a problems with Nginx reload / start / stop

```
sudo pkill -f nginx & wait $!

sudo systemctl start nginx

sudo systemctl restart nginx
```

## Enable port 443 (https) in firewall

```
sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --permanent --list-all

sudo firewall-cmd --reload
```

## Check your live website with SSL

(https://<YOUR-REGISTERED-DOMAIN.TLD>/)

(https://www.<YOUR-REGISTERED-DOMAIN.TLD>/)

(https://<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/)

(https://www.<SUBDOMAIN>.<YOUR-REGISTERED-DOMAIN.TLD>/)

## Enable Nginx to start after a server restart if you have not done it yet

```
systemctl enable nginx

systemctl status nginx
```

__HAPPY CODING__
