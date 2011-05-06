---
layout: post
title: Troubleshooting Dynamic DNS via DHCP
---

[This page](http://www.debian-administration.org/article/343/Configuring_Dynamic_DNS__DHCP_on_Debian_Stable) explains how to get a setup working with dynamic DNS and DHCP.

I'm having a problem where my network was system was configured properly at one point, and then when I made some changes to other parts of the system, dynamic DNS via DHCP functionality broke.  Now, when a new client joins the network, all that happens is the regular DHCP address handout.  From what I can tell from the lack of any messages other than DHCP addressing in /var/log/syslog,there is now no attempt from DHCP to update DNS.

I found that after changing the /etc/bind/rndc.key file from 

    key "rndc-key" {
            algorithm hmac-md5;
            secret "xyz...abc==";
    };

to this.

    key "rndc-key" {
            algorithm hmac-md5;
            secret xyz...abc==;
    };

After restarting both DHCP and DNS services, starting up a VM that was [properly configured](http://jeffwelling.github.com/2010/01/02/Debian-dynamic-dns.html) to send a hostname, and watching /var/log/syslog I saw messages indicating things were working properly again, and joy was had by all.

### References

[http://www.debian-administration.org/article/343/Configuring_Dynamic_DNS__DHCP_on_Debian_Stable](http://www.debian-administration.org/article/343/Configuring_Dynamic_DNS__DHCP_on_Debian_Stable)
[http://brunogirin.blogspot.com/2007/11/dhcp-and-dynamic-dns-on-ubuntu-server.html](http://brunogirin.blogspot.com/2007/11/dhcp-and-dynamic-dns-on-ubuntu-server.html)
[http://jeffwelling.github.com/2010/01/02/Debian-dynamic-dns.html](http://jeffwelling.github.com/2010/01/02/Debian-dynamic-dns.html)
