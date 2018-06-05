---
layout: post
title: Upgrading Fedora 27 Nextcloud Server to Fedora 28.
category: Nextcloud
tags:
- nextcloud
- fedora
subtitle: Fedora 28 Nextcloud Server Upgrade
---

It's time to upgrade our Fedora 27 Nextcloud server (from the previous howto) to Fedora 28. This should be fairly straight forward, but I thought I'd post an update for a few minor gotchas in hopes of saving some people the troubleshooting time.

**Upgrading**

As always, make sure you have good backups of your data. Also make sure your server is already fully updated. Once that is done, we can install the fedora-upgrade package:

`sudo dnf install fedora-upgrade`

Then run:

`sudo fedora-upgrade`

This can take a bit of time and you will be prompted with a few questions throughout the process (you can generally just accept the defaults). Whenever it asks you if you want to do an online or offline upgrade, I've not had any issues doing the online upgrade. This process will take quite a while and will ask you similar questions toward the end of which you can again generally just accept the defaults. You'll then be told to reboot. Once you do so, you'll be running Fedora 28!

**The Minor Gotchas**

At the time of this writing, my upgrades have reverted my services to a disabled status. So you'll need to fire them up and set them to start at boot again:

`systemctl start mariadb`

`systemctl enable mariadb`

`systemctl start httpd`

`systemctl enable httpd`

`systemctl start redis`

`systemctl enable redis`

and afterward it doesn't hurt to run:

`systemctl restart php-fpm` for good measure.

Once you've done these steps, not only should your Nextcloud server be back in business, but also running Fedora 28!


