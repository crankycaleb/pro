---
layout: post
title: Upgrading to Nextcloud 14.0.1
category: Nextcloud
tags:
- fedora
- nextcloud
subtitle: Small Bump in the Road
---

With the Nextcloud upgrade to 14.0.1 I had an issue where the web based update would hang and logs just showed a cron job hung when it wasn't at all. There has been some talk in support threads and I'll post what worked for me.

### 1.) Temporarily allow writes to the http context:
If you're running Fedora like me and followed my setup guides, you'll need to temporarily modify SELinux to allow writes to the http context. You'll always need to do this step before updating:
```
setsebool -P httpd_unified on
```

### 2.) Fix the Nextcloud version number:
One issue seems to be with the version that is stated after the upgrade to the initial Nextcloud 14. You'll need to edit your config/config.php in your nextcloud directory and change the 'version' line to say 14.0.0.0. Mine said 14.0.0.19. 

### 3.) Run the occ updater:
Then you'll need to go to the root of your Nextcloud directory and make the occ upgrade script executable:
```
chmod +x occ
```

Next, run the upgrade as apache user:
```
sudo -u apache ./occ upgrade
```

### 4.) Lock things back down:
After that is finished and you're upgraded, you'll want to remove the execute flag:
```
chmod -x occ
```

And lastly, set the http context back to prevent writes:
```
setsebool -P httpd_unified off
```

And there you have it. Fairly simple adjustment to get 14.0.1 running in no time.

