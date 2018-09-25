---
layout: post
title: Nextcloud on Fedora 27 Server with Let's Encrypt.
category: Nextcloud
tags:
- nextcloud
- fedora
subtitle: Fedora Nextcloud Server Howto
---

The end result of this howto *should* (notice a lot of techies don't speak in absolutes!) give you a fully functional Fedora 27 Nextcloud (13.0.1 at this writing) Server with a sane security baseline. This setup uses MariaDB, Apache with php7-fpm, Redis (for memcache), and Let's Encrypt for ssl sanity.

### 1.) Important Notes for the Fedora Server OS Install:

Whenever you install Fedora Server, select the minimal install **only**. Do **not** select any of the other predefined bundles or packages or you will be in a world of hurt following this howto later. Select the custom server install only. You should also set a static ip and dns name during the install so that much can be ready to go. As far as partitioning, some people will just give everything (other than boot and swap) to the root partition (/). If that isn't your preference, then it is important to note that this howto will have your Nextcloud data directory exist *outside* of the web directories, so outside of /var (I use /data as a mountpoint in this howto). Since that will be the brunt of your data you'll want to plan accordingly with your space and mountpoints.
****
### 2.) First Boot:

Let's go ahead and get our new server updated:

```
dnf update -y
```

Now let's install a lot of the stuff we're going to need for this setup:

```
dnf install -y httpd mariadb mariadb-server php php-gd php-pdo php-pear php-mbstring php-xml php-pear-Net-Curl php-mcrypt php-intl php-ldap php-smbclient php-imap php-mysqli php-pear-MDB2 php-pear-MDB2-Driver-mysqli php-pecl-zip bzip2 policycoreutils-python-utils redis php-pecl-redis wget php-opcache libreoffice certbot python2-certbot-apache mod_ssl tar certbot-apache nano
```

I like nano for a text editor, but you can substitute that package for the text editor of your choice.

There will likely be a kernel update that requires a reboot, so go ahead and do that after the updates are finished:

```
sudo reboot
```
****
### 3.) Start Putting Things Into Place:

After things come back up and you get logged back in, let's create the place where Nextcloud is going to live:

```
mkdir -p /var/www/html/nextcloud
```

Get Nextcloud:

```
wget https://download.nextcloud.com/server/releases/nextcloud-13.0.1.tar.bz2
```

Extract Nextcloud:

```
tar xvf nextcloud-13.0.1.tar.bz2 -C /var/www/html
```

Create the data directory:

```
mkdir -p /data
```

Grab the virtual host file from here:

```
wget -O /etc/httpd/conf.d/nextcloud.conf https://raw.githubusercontent.com/crankycaleb/scripts/master/nextcloud.conf
```

Then set ownership of everything to apache:

```
chown apache:apache -R /var/www/html/nextcloud

chown apache:apache -R /data
```
****
### 4.) Open Up the Firewall:

Let's poke some holes for http and https:

```
firewall-cmd --zone=public --add-port=http/tcp --permanent

firewall-cmd --zone=public --add-port=https/tcp --permanent

firewall-cmd --reload
```
****
### 5.) Get a Database Ready:

Start mariadb and set it to start at boot:

```
systemctl start mariadb

systemctl enable mariadb
```

Do the same for redis:

```
systemctl start redis

systemctl enable redis
```

Create the nextcloud database and secure mariadb. Change `ncuser`, `ncuserpassword`, and `somesecurepassword` to something secure and keep up with it:

```
mysql -e "CREATE DATABASE nextcloud;"

mysql -e "CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'ncuserpassword';"

mysql -e "GRANT ALL ON nextcloud.* TO 'ncuser'@'localhost';"

mysql -e "FLUSH PRIVILEGES;"

mysql -e "UPDATE mysql.user SET Password=PASSWORD('somesecurepassword') WHERE User='root';"

mysql -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');"

mysql -e "DELETE FROM mysql.user WHERE User='';"

mysql -e "DROP DATABASE test;"

mysql -e "FLUSH PRIVILEGES;"
```
****
### 6.) Configure SELinux Permissions:

Let SELinux know what we're doing and make exclusions accordingly. I have a script for this:

```
wget -O ~/nextcloud_selinux.sh https://raw.githubusercontent.com/crankycaleb/scripts/master/nextcloud_selinux.sh

chmod +x ~/nextcloud_selinux.sh
```

Execute the script (will take several seconds):

```
~/nextcloud_selinux.sh
```
****
### 7.) Configure Web Services:

Start apache and set it to run at boot:

```
systemctl restart httpd

systemctl enable httpd
```

Update the `php-opcache` ini file:

```sed -i -e 's/;opcache.enable_cli=0/opcache.enable_cli=1/' /etc/php.d/10-opcache.ini;

sed -i -e 's/opcache.max_accelerated_files=4000/opcache.max_accelerated_files=10000/' /etc/php.d/10-opcache.ini;

sed -i -e 's/;opcache.save_comments=1/opcache.save_comments=1/' /etc/php.d/10-opcache.ini;

sed -i -e 's/;opcache.revalidate_freq=2/opcache.revalidate_freq=1/' /etc/php.d/10-opcache.ini;
```

Restart `php-fpm` to apply opcache settings:

```
systemctl restart php-fpm
```
****
### 8.) Important Notes About DNS:

Creating a DNS entry is optional, but when the Nextcloud first run wizard happens in the browser, it sets the config.php to trust the URL in the browser. If you do not have DNS setup yet, you can change this in your config.php later but I highly recommend just setting your host file on the client machine you're using to resolve to what *will* be the fqdn of the finished product. After you've taken care of that, go to the url in your web browser and append /nextcloud to it. For now, we'll be http:// so that url would be something like http://myfqdn.org/nextcloud.

So now you should see the admin account creation page in your browser. go ahead and do that:

![step1](/img/fedoranextcloud/nextcloudcreate.png)

Then click the storage and database dropdown:

![step2](/img/fedoranextcloud/nextcloudstorage.png)

Change the data folder to /data:

![step3](/img/fedoranextcloud/nextclouddata.png)

Change the database to MySQL/MariaDB:

![step4](/img/fedoranextcloud/nextclouddb1.png)

Then fill out with the information you setup earlier:

![step5](/img/fedoranextcloud/nextclouddb2.png)

Then click the finish setup button:

![step6](/img/fedoranextcloud/nextcloudfinish.png)

It will take a little bit to get everything set in place and then you'll be logged in live to your new nextcloud server!

Now go back to your SSH session and update the Nextcloud config.php file to tell it to use redis for the memory cache and file locking:

```
nano /var/www/html/nextcloud/config/config.php
```
and add this toward the bottom:

```
'memcache.locking' => '\OC\Memcache\Redis',

'memcache.local' => '\OC\Memcache\Redis',

    'redis' => array(

    'host' => 'localhost',

    'port' => 6379,

),
```

Then restart apache:

```
systemctl restart httpd
```

The hard stuff is done!
****
### 9.) Switch Document Root to Your Nextcloud Install:

These edits will remove the need to append /nextcloud to your url:

```
nano /etc/httpd/conf/httpd.conf
```

Then change:

```
DocumentRoot "/var/www/html"
```

to: 

```
DocumentRoot "/var/www/html/nextcloud"
```

We'll also need to update the Nextcloud config.php with the new url:

```
nano /var/www/html/nextcloud/config/config.php
```

```
'overwrite.cli.url' => 'http://fqdn.org',
```

And finally, restart apache:

```
systemctl restart httpd
```

You should now be able to browse and access your nextcloud via the base url.
****
### 10.) Cron Setup:

You're going to want cron to handle some things in the background for you depending upon various apps you may install in nextcloud. The job will need to run as the apache user:

```
crontab -u apache -e
```

then add:

```
*/15  *  *  *  * php -f /var/www/html/nextcloud/cron.php
```

to exit, type :wq and press enter. You can verify if the cron job has been added and scheduled by executing:

```
crontab -u apache -l
```

Next lets go to our nextcloud web interface settings --> basic settings under administration and switch from Ajax to cron:

![step1](/img/fedoranextcloud/nextcloudcron.png)

You can come back here and make sure the job is running now since the "Last job ran" should never be greater than 15 minutes.
****
### 11.) SSL Setup with Let's Encrypt:

Now that we have a functional nextcloud server, we need to secure it. This will require your DNS to be resolveable from the outside world, and port 80 forwarded as well as 443 since certbot uses tcp 80 to confirm your ownership. Be sure to replace with your own info:

```
certbot --apache certonly --email youremail@domain.com --domain fqdn.org --agree-tos --non-interactive
```

Now we update apache to look for the cert files:

```
nano /etc/httpd/conf.d/ssl.conf
```

replace:

```
SSLCertificateFile /etc/pki/tls/certs/localhost.crt
```

with:

```
SSLCertificateFile /etc/letsencrypt/live/fqdn.org/cert.pem
```

and:

```
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
```

with:

```
SSLCertificateKeyFile /etc/letsencrypt/live/fqdn.org/privkey.pem
```

and finally replace (you may also need to uncomment this line):

```
SSLCertificateChainFile /etc/pki/tls/certs/server-chain.crt
```

with:

```
SSLCertificateChainFile /etc/letsencrypt/live/fqdn.org/chain.pem
```

Then restart apache:

```
systemctl restart httpd
```

Now refresh or browse to your domain with https:// and you should be live with a good certificate!
****
### 12.) Enable HTTP Strict Transport Security:

Now that we know we have a working cert, configure HSTS to help prevent man in the middle attacks:

```
nano /etc/httpd/conf.d/ssl.conf
```

Find `<VirtualHost _default_:443>` and add the following line underneath:

```
Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
```

Restart httpd again:

```
systemctl restart httpd
```

and now your security warnings for HSTS should be cleared up with "All checks passed" on the basic settings section of the administrative settings pane.
****
### 13.) Disable Header Info Leak:

With security in mind, it is also a good idea to conceal our server version banner. Just add these two lines to your httpd.conf and restart httpd again:

```
ServerTokens Prod
```

and:

```
ServerSignature Off
```

You can test before and after at https://tools.geekflare.com/tools/http-header-test.

Now you have a fully functional and decently secured nextcloud server of your own! Documentation for the nextcloud project is great, so go dig around in the apps and find the next steps for customizing your setup.
****
### Future Updates:

Before running Nextcloud updates in the future, you'll need to ssh to your server and temporarily loosen the write restrictions for selinux beforehand. If you don't, the web updater will error out with permissions errors. Simply run:

```
setsebool -P  httpd_unified  on
```

and when updates are finished, be sure to lock things back down with:

```
setsebool -P  httpd_unified  off
```
