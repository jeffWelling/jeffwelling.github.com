---  
layout: post  

title: How to commit with Rugged
---  

So far in my brief use, the Rugged gem seems to be a delightfully fast way of using git in Ruby, I just have one gripe (and it's a petty one): The interface takes getting used to and requires intimate knowledge of the inner workings of git.

Rugged is still in development, so this blog may be out of date by the time you read it, you should definitely check with the lates documentation for both Rugged and libgit2 (if there is a discrepency in the documentation of one project about how the other project works, go with the documentation that goes with the project). 

1) Add the files to the index.  
2) Write the index.  
3) Create the commit.  
4) Save the commit.  

Which can be done like this;  

    pry$ require 'rugged'  
    pry$ repo = Rugged::Repository.new repository_path  
    pry$ index = repo.index  
    pry$ index.add 'second_fakefile'  
    pry$ tree= repo.lookup( repo.index.write_tree )    #At this point, the files are staged
    pry$ author={ :email=>'jeff.welling@gmail.com', :time=>Time.now, :name=>'Jeff Welling' }  
    pry$ parents=[ repo.lookup( repo.head.target ).oid ]  

It was at this point that I ran into trouble.  `index.write_tree` would succeed, and would return the sha of the tree that was written in string form, and I could verify this by using `git status` to see that the file was indeed staged and ready to be committed. When I tried to commit though, I got an error;  

    pry$ x=Rugged::Commit.create( 
          repo, 
          :author=>author, 
          :message=>"Hello world\n\n", 
          :committer=>author, 
          :parents=>parents, 
          :tree=>tree.oid,
          :update_ref=>'HEAD'  )
    TypeError: wrong argument type nil (expected String)

I double-checked my usage against the documentation from the README on the Rugged Github page, but even after asking for help in the #libgit2 irc channel on Freenode and [posting a question on Stackoverflow][0] I wasn't getting anywhere.  One of (well, the only) answer I got to the Stackoverflow asked what code I was running which lead me to wonder if it was a version discrepency.  I checked the Rugged Rubygems page and though it said the main version was 0.16.0, the version I was running, there was a prerelease version available with version number 0.17.0.b6.  I wasn't aware that Rubygems did prereleases, but after learning about the --prerelease option to the `gem` command I did `gem install --prerelease rugged` and that installed the version 0.17.0.b6. After installing that prerelease gem I was able to do the commit like normal.

Note: The `:update_ref` option to the Commit.create command isn't documented in the README on the Rugged Github page, but if it documented in the comments in the `ext/rugged_commit.c` file in the Github repository, and I've tested using it.  The catch is that without that, the commit will be created and written to the ODB but no reference will be updated so it will appear to most people that Commit.create is broken because they can't see it on any of their branches.  

Note: To get the sha of an object, the method you're looking for is `obj.oid`.  

And so that is the story of how I learned to do commits in Rugged. I hope this has been useful or at least an educational read for you.

I went back after the fact and ran the tests that came with 0.16.0 and many of the tests relating to creating and writing commits failed, so it's no wonder I was having trouble.  Hopefully 0.17.0 has been published as non-prerelease by the time you come to using Rugged.

References:  
0) [http://stackoverflow.com/questions/12649697/how-to-commit-with-ruby-bindings-for-libgit2/12651234][0]  
1) [http://librelist.com/browser//libgit2/2011/2/19/initing-a-repository-adding-files-to-the-index-and-committing/#d94ce8df18ff0202ce904180286a4a85][1]    
2) [https://github.com/libgit2/rugged/blob/development/ext/rugged/rugged_commit.c][2]  
3) [https://github.com/libgit2/rugged][3]  
4) [http://rubydoc.info/gems/rugged/0.16.0/frames][4]  


[0]: http://stackoverflow.com/questions/12649697/how-to-commit-with-ruby-bindings-for-libgit2/12651234
[1]: http://librelist.com/browser//libgit2/2011/2/19/initing-a-repository-adding-files-to-the-index-and-committing/#d94ce8df18ff0202ce904180286a4a85
[2]: https://github.com/libgit2/rugged/blob/development/ext/rugged/rugged_commit.c
[3]: https://github.com/libgit2/rugged
[4]: http://rubydoc.info/gems/rugged/0.16.0/frames
