---  
layout: post  

title: The Git Problem That Wasnt
---  


Earlier today I ran into a strange problem that I, at the time, was attributing to using two different versions of git on two different machines. The problem that I was having was that when I tried to do a git pull via ssh from my development VM, I was getting this error:

    jeff@stan:~/Documents/Projects/tget$ git fetch cnd
    jeff@cnd's password: 
    fatal: protocol error: bad line length character: GPG_

My first instinct was to take my problem before the great almighty (though recently diminishing in greatness…) Google, but the results I got weren't very helpful and didn't include the "GPG_" part of the string -- which should have been my first hint.

It was at this point that I gave up on tackling this specific problem in the interest of progressing on the task at hand, so I put it on the back-burner in my mind.

An hour or so later while copying .bashrc and related files to my desktop, I finally made the connection: the problem was related to having recently installed Gnu-PG and gpg-agent on my development VM. I use it to sign the tags for the releases of my programs, but unbeknownst to me I had misconfigured it due to a lack of understanding of the [finer details](http://hacktux.com/bash/bashrc/bash_profile) of the `.bashrc` and `.bash_profile` files.  After fixing that mistake, I was successfully able to `git fetch cnd`.  
And cake was had by all!
