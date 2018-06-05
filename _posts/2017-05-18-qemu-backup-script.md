---
layout: post
title: Backing Up QEMU VM's
category: Virtualization
tags:
- qemu
- backup
subtitle: The beginnings of a functional script
---

I'd like to document more about QEMU and the robust virtualization environments you can build using opensource, but today I want to post a simple QEMU backup script that I've had a lot of success with. Hopefully someone else will stumble upon this and be able to use it for starting out, or even submit improvements to the script in my github repo.

My comments will be inside the script itself, that way as I update it and make changes people will always see the freshest comments rather than a stale post. You can find the script in my repo [here](https://github.com/crankycaleb/scripts/blob/master/QEMUbackuptemplate.sh).

As always, not responsible for anything that happens! 
