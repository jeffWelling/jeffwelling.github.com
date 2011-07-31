---
layout: post
title: Better booting
---

So I've run into this problem on my Debian 5.0 system, as many other people have, involving many storage disks and the nondeterministic assignment of `/dev/sd...` names at boot time. The problem is documented in many places all over the web, as are many documented solutions, so I won't go into unnecessary detail describing the problem or every possible solution. What I intend to document here is the solution that I chose in a clear and concise way, I've found that many of the pages with solutions aren't formatted as clearly as they could have been even if they were ultimately helpful in solving the problem.

To add a label to a partition\[[3\]]\[[4]\], use

    root@mine:~# tune2fs -Lroot /dev/sda1

Though for some reason, the label does not become visible in my partition list when I run `cfdisk /dev/sda` as \[[3]\] says it's supposed to.  [One page][5] I found demonstrates that one way of checking that a label has indeed been added is to reboot and then do

    ls -l /dev/disk/by-label/

Which, on my system, shows

    root@mine:~# ls -l /dev/disk/by-label
    total 0
    drwxr-xr-x 2 root root  80 2011-07-31 02:23 .
    drwxr-xr-x 6 root root 120 2011-07-31 02:23 ..
    lrwxrwxrwx 1 root root   9 2011-07-31 02:23 media -> ../../md0
    lrwxrwxrwx 1 root root  10 2011-07-31 02:23 root -> ../../sda1 

After adding a label for root, it's time to add one for swap as outlined [here][2].

    # swapoff /dev/sda2
    # mkswap -L Swap /dev/sda2
    Setting up swapspace version 1, size = 1998737 kB
    LABEL=Swap, UUID=7cdfeb21-613b-4588-abb5-9d4049854e9a
    # swapon /dev/sda2

Don't forget to change `/dev/sda2` to whatever your swap is, and please *please* doublecheck your device names before executing, nobody wants you to wipe your / drive.

Then, remembering that these instructions worked on Grub version 0.97 (not Grub2), change your `/etc/fstab` file to resemble this

    # <file system> <mount point>   <type>  <options>       <dump>  <pass>
    proc            /proc           proc    defaults        0       0
    #/dev/sda1       /               ext3    errors=remount-ro 0       1
    LABEL=root       /               ext3    errors=remount-ro 0       1
    LABEL=media     /mnt2           ext3    errors=remount-ro 0        0
    LABEL=swap       none            swap    sw              0       0
    /dev/scd0       /media/cdrom0   udf,iso9660 user,noauto     0       0

Then the last step is to edit `/boot/grub/menu.lst` to resemble this

    title           Xen 3.0.3-1-amd64 / Debian GNU/Linux, kernel 2.6.18-4-xen-amd64
    root            (hd0,0)
    kernel          /boot/xen-3.0.3-1-amd64.gz
    module          /boot/vmlinuz-2.6.18-4-xen-amd64 root=LABEL=root ro acpi=off noapic console=tty0
    module          /boot/initrd.img-2.6.18-4-xen-amd64
    savedefault


And thats it.  Now, I haven't yet figured out a way to reliably reproduce the bug that causes the nondeterministic device names to get shuffled so I'm unable to verify that this *actually* works, but since implementing this I haven't had the bug happen to me.  If that changes, I will update this blog post accordingly.

### References


[1]: http://debian-resources.org/node/9/ "How to mount disks by label"
[2]: http://wiki.debian.org/Part-UUID "UUIDs, Labels, and fstab"
[3]: http://www.debian-administration.org/articles/522 "Mounting file-systems by label rather than device name"
[4]: http://webcache.googleusercontent.com/search?q=cache:peBdLm0LTFQJ:www.debian-administration.org/articles/522&hl=en&strip=1 "Google Cache of 'Mounting file-systems by label rather than device name'" 
[5]: http://wiki.debian.org/Part-UUID "Part-UUID"
