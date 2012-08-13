---  
layout: post  

title: Switching To The Rugged Gem
---  

It's one of those really annoying problems that can't be solved with a one-liner change. Not even a small code refactor would fix it. It's a speed problem, the program is taking too long to load, far longer than it should take. It must be fixed!

After adding some debug output lines to the code, the speed problem seems to be tracked down to lib/ticgit-ng/base.rb file on line 40 in commit d45551213719d8f42ef43208bdb497c00a85b7f8. Now, this line doesn't inherently require switching to the Rugged gem, it could be written to directly read the .git/re
fs/heads/ directory to retrieve a list of the branch names, bypassing the calls to the git gem and saving time, but A) that would be a dirty hack, B) I need to switch over to the Rugged gem anyway and doing so now may solve both problems at the same time.

So, I need to get familiar enough with Rugged to be able to switch to it. Rugged appears to be written very differently from the Git gem in that the Git gem tries to emulate the user interface to git but in an API, whereas the Rugged gem is a Ruby implementation of the Git API.

I'll keep a micro-blog of how I managed.

9:29pm -- [This] [0] page documents most if not all of the libgit2 methods, with accompanying code examples.  Given that Rugged is supposed to be a Ruby interface to libgit2 this seems like a good place to start.

10:36pm -- After trying to install the Rugged gem to play around a bit and running into trouble, I think the best thing to do will be to use the dirty hack for now (manually read .git/refs/heads/) to fix the speed problem. I do intend to eventually switch TicGit-ng over to the Rugged gem, but for a patch that switches to Rugged to be applied and pushed I need the Rugged installation to be seamless and at present, due to still being in heavy development, both Rugged and libgit2 are not seamless to install. I will resolve my installation problem with Rugged and continue playing with that separately and will try and get a head start on the patch now, but for the immediate solution for the speed problem I will have to go with the dirty hack.

3:08am -- I've spent a while now trying to speed up the code and I've found a couple different areas that I was able to improve that should help things, I'm just running some tests to be able to compare the differences in speed.

And here it is, the results of the speed test (test.rb ran `ti list` 300 times);

    $ time ./test.rb; git checkout run_faster; time ./test.rb

    real    6m36.835s
    user    2m40.766s
    sys     2m35.706s
    Switched to branch 'run_faster'

    real    4m17.112s
    user    2m2.988s
    sys     1m22.889s

That's a fairly decent speedup, and it's definitely noticeable when calling the command as well. All in all, mission accomplished for now.

[0]: http://libgit2.github.com/libgit2/ex/HEAD/general.html
