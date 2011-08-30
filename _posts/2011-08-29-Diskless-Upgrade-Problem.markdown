---
layout: post
title: Diskless Upgrade Problem
---

After following [this guide][1] to get my diskless Ubuntu system up and running, I ran into a problem when I tried to upgrade.  Basically, update-grub was failing because there was no disk. The problem was that this failure caused upgrades to the kernel and other system components to fail for no good reason (HD was intentionally removed).

I remember I had come across this problem and had solved it once before but couldn't remember how off the top of my head, so my first tactic was Google.  But after a good fifteen minutes of searching with various combinations ofvarious keywords I couldn't find anything particularly useful aside from a page that suggested uninstalling grub altogether, so I decided to try another tactic.  This time I went and found the script that was failing and commented out the command, in this case, `exec update-grub` in the file `/etc/kernel/postinst.d/zz-update-grub`.

It's a small file, and the line that needed commenting out was only line 15.After commenting that line out, I was able to complete my upgrades.

[1]: https://help.ubuntu.com/community/DisklessUbuntuHowto "Diskless Ubuntu Howto"
