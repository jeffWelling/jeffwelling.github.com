---
layout: post
title: Debian Git Completion
---

I run [Debian](http://www.debian.org/intro/about#what) stable, and right now thats 5.0 Lenny.  I also use git, and would like it if git completion would work.

First, I added this to my `~/.profile` file;

    #For git tab completion and the special prompt thingy.
    source /usr/local/git/contrib/completion/git-completion.bash
    GIT_PS1_SHOWDIRTYSTATE=true
    export PS1='[\u@CnD \w$(__git_ps1)]\$ '
 
This adds the name of the branch to your bash prompt.  Thing is though, I was running into this error;

    bash: /usr/local/git/contrib/completion/git-completion.bash: No such file or directory
    bash: __git_ps1: command not found

And I didn't know why.  Well, it turns out that the second line that I added to my `~/.profile` file was wrong, the file `/usr/local/git/contrib/completion/git-completion.bash` doesn't exist in that place on Debian.  This could be due to any number of reasons, including but not limited to that bit of code was maybe written for another system on which that file does exist.  But the solution was easy, just update the `source` line to point to the new file.  Some digging showed me where this was.  So, my `~/.profile` file now looks like this;

    #For git tab completion and the special prompt thingy.
    source /etc/bash_completion.d/git
    GIT_PS1_SHOWDIRTYSTATE=true
    export PS1='[\u@CnD \w$(__git_ps1)]\$ '

And now I have a working tab-completing prompt which shows my current branch.
