---
layout: post
title: Apt Pinning
---

[Apt Pinning](http://jaqque.sbih.org/kplug/apt-pinning.html) is a great way of installing packages from testing or unstable on your stable system.  This is especially useful when I'm working on a project that requires a newer version of a library than is currently in Debian Stable, but has too many uses to enumerate here.

If you'll notice though, the Apt Pinning page that comes up at the top of Google searches is somewhat lacking in detail in some aspects.

For example, the examples given for sources.list require changes to actually be useful to anyone today because things have changed.  The original author provided two examples, the first one;

    #Stable
    deb http://ftp.us.debian.org/debian stable main non-free contrib
    deb http://non-us.debian.org/debian-non-US stable/non-US main contrib non-free

    #Testing
    deb http://ftp.us.debian.org/debian testing main non-free contrib
    deb http://non-us.debian.org/debian-non-US testing/non-US main contrib non-free

    #Unstable
    deb http://ftp.us.debian.org/debian unstable main non-free contrib
    deb http://non-us.debian.org/debian-non-US unstable/non-US main contrib non-free

and a [second one](http://jaqque.sbih.org/kplug/sources.list).


The first example is odd because first of all, it doesn't include a line for security.debian.org, and second it has 2 lines for each version - to my understanding this is superfluous.  You only _need_ one line for each of Stable Testing and Unstable, but more can be added.

The problem with the second example is there are so many extraneous lines, and it includes lines that if included today wouldn't work, and some lines that should be explained so that they can be improved on for international users.  You'll notice in the second example, as well as in my example below that the URLs like `ftp.ca.debian.org` and `ftp.us.debian.org` have a pattern, the 2 letter country code in between `ftp.` and `debian.org`.  This can be changed to your local country code to use a local mirror instead of a mirror from Canada or the US.

This is my sources.list file.  I use main contrib and non-free, but if you just wanted to use main or main and contrib you can change it accordingly.

    deb http://ftp.ca.debian.org/debian/ stable main contrib non-free
    deb http://security.debian.org/ stable/updates main contrib non-free
    deb-src http://ftp.ca.debian.org/debian/ stable main contrib non-free
    deb-src http://security.debian.org/ stable/updates main contrib non-free

    deb http://volatile.debian.org/debian-volatile stable/volatile main contrib non-free
    deb-src http://volatile.debian.org/debian-volatile stable/volatile main contrib non-free

    deb http://ftp.ca.debian.org/debian/ testing main contrib non-free
    deb http://security.debian.org/ testing/updates main contrib non-free
    deb-src http://ftp.ca.debian.org/debian/ testing main contrib non-free
    deb-src http://security.debian.org/ testing/updates main contrib non-free

    deb http://ftp.ca.debian.org/debian/ unstable main contrib non-free
    deb-src http://ftp.ca.debian.org/debian/ unstable main contrib non-free

To setup apt-pinning on your Debian system copy the above and overwrite it onto your old `/etc/apt/sources.list` file.  Then copy this;

    Package: *
    Pin: release a=stable
    Pin-Priority: 700

    Package: *
    Pin: release a=testing
    Pin-Priority: 650

    Package: *
    Pin: release a=unstable
    Pin-Priority: 600

And paste it into a file which may not exist yet called `/etc/apt/preferences`.  This enables apt pinning and sets it so that stable is the branch that is used when no branch is specified when calling apt-get or aptitude.


Theres two ways two install packages;

    aptitude install <package>/unstable

Will install <package> from unstable trying to meet dependencies from stable, and this may not always work which is why I use;

    aptitude -t unstable install <package>

Which will install package and all dependencies from unstable.
