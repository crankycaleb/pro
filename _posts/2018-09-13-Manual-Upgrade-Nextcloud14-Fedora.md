---
layout: post
title: Manually Upgrade to Nextcloud 14
category: Nextcloud
tags:
- fedora
- nextcloud
subtitle: Sometimes You Gotta Get Your Hands Dirty
---

Nextcloud has one of the best track records I've ever experienced with next to flawless and easy service upgrades, but sometimes I just like to clean things up a bit myself. I had an issue where the updater section just *refused* to show up at all in the administration section of my Nextcloud 13 instance. This was almost certainly my own fault due to some experimentation I had done throughout the life of my server, so it was time for a manual upgrade. Nextcloud does staggered release rollouts for their update utility, that way if there happens to be an issue they can stop it to get that fixed before it hits everyone (I don't know that they've ever had to stop one). So this is also a handy way to "jump the gun" and get your server upgraded sooner rather than days later. If you're *not* in my situation and can access the update gui, you can also temporarily switch to the "Beta" channel to trigger the upgrade availability and then switch back to "Stable" afterward. Here's my notes for the manual upgrade that should help you when all else fails and things go south.

### 1.) Temporarily allow writes to the http context:

If you're running Fedora like me and followed my setup guides, you'll need to temporarily modify SELinux to allow writes to the http context. You'll need to do this step even if you're using the web based updater rather than the manual way:

```
setsebool -P httpd_unified on
```
****
### 2.) Shutdown apache and backup nextcloud:

Another good reason to always have your data directory housed *outside* of your nextcloud install is for much simpler backups and upgrades. If you followed my howto then that is how you are already setup. If you're running the ownbackup app (I highly recommend that) you also already have your database backed up to your data directory.

So let's first shutdown apache:

```
sudo systemctl stop httpd
```

Then *move* our nextcloud dir to be a backup directory instead.

```
mv /var/www/html/nextcloud /var/www/html/nextcloud-backup
```
****
### 3.) Download the latest nextcloud package and move things into place:

As of this writing, the latest version is 14. Let's grab it and make sure we're still outside of our web directory, like in our home directory:

```
wget https://download.nextcloud.com/server/releases/nextcloud-14.0.0.zip
```

Then unzip:

```
unzip nextcloud-14.0.0.zip
```

Then move into place:

```
mv nextcloud /var/www/html/
```

Now we need to grab our config.php from our backup and place it in the new install:

```
cp /var/www/html/nextcloud-backup/config/config.php /var/www/html/nextcloud/config/
```

At this point, you'll also want to compare the apps directory of your backup with the apps directory of the new install and probably copy those over as well. You'll still need to enable them after the upgrade. Your data is still safe in your database, so you can also just browse the app store and add them again fresh if you prefer.
****
### 4.) Fix the permissions:

Now we'll want to get our permissions on all the shuffling we've done straightened out. Go ahead and cd to /var/www/html if you aren't already there and then do the following:

```
chown -R apache:apache nextcloud
```

Then:

```
find nextcloud/ -type d -exec chmod 750 {} \;
```

and finally:

```
find nextcloud/ -type f -exec chmod 640 {} \;
```
****
### 5.) Bringing things back up:

Start apache:

```
sudo systemctl start httpd
```

And finally, launch the actual upgrade process. This may take a few minutes:

```
sudo -u www-data php occ upgrade
```
****
### 6.) Finishing up:

After the upgrade is complete, you're ready to go checkout your instance and the settings page to make sure everything is there. Then you'll have the usual process of going in and enabling your apps again. Don't forget to lock down with SELinux again:

```
setsebool -P httpd_unified off
```

When I checked my settings admin page after these steps, my autoupdate section gui was back! So now for future upgrades I can just do `setsebool -P httpd_unified on` from the terminal, use the handy graphical upgrade process from the browser, and then lock back down with `setsebool -P httpd_unified off`
****
### Extra points - lock down your referrer-policy a bit more:

After upgrade, you may see an additionaly complaint in your settings check page with a recommendation to lock down your referrer-policy some more. In my case, this is done by editing ssl.conf in /etc/httpd/conf.d/ and adding the following in the `_default_:443` section beneath the existing "Header always set StrictTransport..." line:

```
Header always set Referrer-Policy "strict-origin"
```

Save, restart apache, and refresh your settings page again to see the issue resolved!
