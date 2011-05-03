---
layout: post
title: Diskless Debian Squeeze
---

## Netbooting Debian Squeeze  
This page is about netbooting [Debian](http://www.debian.org/) [squeeze](http://www.debian.org/releases/squeeze/).  That is, using [PXE](https://secure.wikimedia.org/wikipedia/en/wiki/Preboot_Execution_Environment) to boot a [diskless](https://secure.wikimedia.org/wikipedia/en/wiki/Diskless_node) system with `/` (root) on a fileserver mounted via [NFS](https://secure.wikimedia.org/wikipedia/en/wiki/NFS), a method known as NFSRoot.  

The easiest way to do this IMO is to actually install Debian Squeeze to a machine, make the necessary modifications to turn it into a diskless client and then copy the filesystem from that machine to a directory in the NFS share.  Note, this page assumes root privileges.

### First things first.  Install Debian Squeeze to a machine (or virtual machine ) of your choosing.  

### Prerequisites  

**On the server**, you will need these packages   

     apt-get install dhcp3-server  
     apt-get install tftp-hpa  
     apt-get install syslinux  
     apt-get install nfs-kernel-server  

**On the client**, you will need these packages

     apt-get install initramfs-tools

### Configure DHCP server 

   **NOTE:** 
Most networks already have a DHCP server configured, and having more than one of them on the network at a time can cause problems for other computers on the network. Make sure you disable any other DHCP servers if you are starting a new one, instead of changing the configuration for the existing server.  

Now, configure the DHCP server.  I prefer vim,

    `vim /etc/dhcp3/dhcpd.conf` 

You need to add a `next-server` and `filename` options, and the resulting file should look something like this.  

    default-lease-time 600;  
    max-lease-time 7200;  
    option domain-name "xephon";  
    option domain-name-servers 192.168.1.1, 192.0.0.1, 194.2.0.50;  
    option routers 192.168.1.1;  
    subnet 192.168.1.0 netmask 255.255.255.0 {  
    range 192.168.1.50 192.168.1.99;  
      next-server 192.168.1.1;
      filename "/tftpboot/pxelinux.0";  
    }  

The `next-server` option tells the client which IP to ask for the file named in `filename`.  
You can see the **Set up DHCP** section of the [this page](http://wiki.linuxquestions.org/wiki/Diskless_Workstation) for an example of setting up a static client section.

### Set up NFS  
First, we will create the directory that will contain our nfs shares, and then a directory for the root of the client who we will name zim.  
  
    mkdir /nfs  
    mkdir /nfs/zim  

Don't forget to export them,

`vim /etc/exports`  

and add this line

`/nfs/zim 192.168.1.0/255.255.255.0(rw,async,no_root_squash,no_subtree_check)`

This will allow anyone in the network of 192.168.1.0/24 to mount this share.  If this is not what you want, you can restrict it to a single IP such as `192.168.1.101/32`, or simply `*` to allow anyone access. Then, reload the file with,  

`exportfs -rv`  

### Configure the client  

Let's start with `/etc/fstab`.  Edit it so that your current root and swap are commented out, and add the necessary lines, both of which are shown in this example.  

    ###/dev/hda1       /               ext3    noatime,errors=remount-ro 0       1
    ###/dev/hda5       /mnt/hda5       ext3    noatime         0       2
    ###/dev/hda6       none            swap    sw              0       0

    /dev/nfs        /               nfs     defaults 0 0
    none            /tmp            tmpfs   defaults 0 0
    none            /var/run        tmpfs   defaults 0 0
    none            /var/lock       tmpfs   defaults 0 0
    none            /var/tmp        tmpfs   defaults 0 0
    none            /media          tmpfs   defaults 0 0

Next, we need to edit `/etc/initramfs-tools/initramfs.conf`.  Edit it so the `BOOT` and `MODULES` lines are as shown  

    BOOT=nfs
    MODULES=netboot

Now, create a new initrd image.

    mkinitramfs -o initrd.img.netboot
    mv initrd.img.netboot /boot/.

Now, don't forget to edit the `/etc/network/interfaces` file. It needs to look like this.  

    # The primary network interface  
    #allow-hotplug eth0  
    iface eth0 inet manual  

Then, mount the NFS share and begin copying over the filesystem to the server.  

    mount -tnfs -onolock 192.168.1.4:/nfs/zim /mnt   
    cp -axv /. /mnt/.  
    cp -axv /dev/. /mnt/dev/.  

And finally, back on the server, configure tftp.  Remember that in Debian Squeeze, for the tftp-hpa package,
the tftp folder is in `/var/lib/tftpboot`.  

    mkdir -p /var/lib/tftpboot/pxelinux.cfg  
    cp /usr/lib/syslinux/pxelinux.0 /var/lib/tftpboot/.  
    cp /nfs/zim/boot/vmlinuz<TAB> /var/lib/tftpboot/vmlinuz-zim  
    cp /nfs/zim/boot/initrd.img.netboot /var/lib/tftpboot/initrd.img.zim  

Where you see <TAB>, press the TAB key and it will automagically fill in the kernel version, assuming you only have one version in there.  
Now, create a PXE config file

    vim /var/lib/tftpboot/pxelinux.cfg/default

and add contents such as these, replacing the IP address and kernel version with the one your using.

    LABEL linux
    KERNEL vmlinuz-2.6.15-1-486
    APPEND root=/dev/nfs initrd=initrd.img.netboot nfsroot=192.168.1.4:/mnt/hda5/yuki ip=dhcp rw

### Enjoy

Now, you should have a working thinclient.  
Don't forget the changes you've made to the existing machine, in it's current state it isn't bootable, but that isn't a problem because we assume you've installed it to a machine that you will be using as a thinclient.  Simply revert the changes to put the machine back in a bootable state.

### References

http://wiki.linuxquestions.org/wiki/Diskless_Workstation  
https://help.ubuntu.com/community/DisklessUbuntuHowto  

