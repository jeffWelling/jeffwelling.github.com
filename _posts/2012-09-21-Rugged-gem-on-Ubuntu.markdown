---
layout: post
title: Rugged gem on Ubuntu
---

I tried to install the Rugged gem today on my Ubuntu 12.04 system and I got this error,

    $ gem install -r rugged
    Building native extensions.  This could take a while...
    ERROR:  Error installing rugged:
      ERROR: Failed to build gem native extension.

            /usr/bin/ruby1.8 extconf.rb
     -- tar zxvf libgit2-dist.tar.gz
     -- make -f Makefile.embed
    checking for main() in -lgit2_embed... yes
    checking for git2.h... no
    ERROR: Failed to build libgit2
    *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of
    necessary libraries and/or headers.  Check the mkmf.log file for more
    details.  You may need configuration options.

    Provided configuration options:
      --with-opt-dir
      --without-opt-dir
      --with-opt-include
      --without-opt-include=${opt-dir}/include
      --with-opt-lib
      --without-opt-lib=${opt-dir}/lib
      --with-make-prog
      --without-make-prog
      --srcdir=.
      --curdir
      --ruby=/usr/bin/ruby1.8
      --with-git2_embedlib
      --without-git2_embedlib


    Gem files will remain installed in /var/lib/gems/1.8/gems/rugged-0.16.0 for inspection.
    Results logged to /var/lib/gems/1.8/gems/rugged-0.16.0/ext/rugged/gem_make.out

It took some digging to find out what was going on, but if you check the bottom of the mkmf.log file you might notice a zlib error;

    /var/lib/gems/1.8/gems/rugged-0.16.0/ext/rugged/vendor/libgit2-dist/include/git2/zlib.h:10:18: fatal error: zlib.h: No such file or directory

Some brief googling will show that this means we need to install the `zlib1g-dev` package,

    $ sudo apt-get install zlib1g-dev

Now you should be able to install Rugged without problems.

    $ sudo apt-get install -r rugged

And happyness was had by all!
