---  
layout: post  

title: How to commit with Rugged
---  

So far in my brief use, the Rugged gem seems to be a delightfully fast way of using git in Ruby, I just have one gripe (and it's a petty one): The interface takes getting used to and requires intimate knowledge of the inner workings of git.

Rugged is still early in development, note that as of this writing it has yet to reach '1.0' and so I expect this will change in the future, but at present in order to use Rugged you don't use a method called 'commit', you need to know what a commit does. 

When you're using Git from the command line, the normal procedure for committing is to `git add foo/bar/myfile.rb` and then to `git commit`. The first command will add `foo/bar/myfile.rb` to what git refers to as its index, files here are referred to as 'staged'. The mechanics of `git add` are a little more nuanced then that, the add command is what actually creates the tree and blob objects, but for the purposes of this brief howto we will glaze over that for now. The commit command takes the tree and blob objects and ties them together in the commit by storing the sha of the base tree in the commit. That tree object points to all of the blobs under it as well as any trees, and by looking up those trees you can get their children and so on, so by storing a reference to that base tree you are storing a reference for the state of your entire repository at that point in time. After the commit object is saved, the head for your current branch is updated to point to the new commit object.

So now that we have a rough idea of how the git command works, you have a rough idea of what you have to do with Rugged in order to do a commit.  

Note: To get the sha of an object, the method you're looking for is `obj.oid`.  

1) Create the necessary tree and blob objects.  
2) Create the commit.  
3) Save the commit.  

    pry$ require 'rugged'  
    pry$ repo = Rugged::Repository.new repository_path  
    pry$ index = repo.index  
    pry$ index.add 'second_fakefile'  
    pry$ tree= repo.lookup( repo.index.write_tree )  
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

And so that is the story of how I learned to do commits in Rugged. I hope this has been useful or at least an educational read for you.

References:
http://librelist.com/browser//libgit2/2011/2/19/initing-a-repository-adding-files-to-the-index-and-committing/#d94ce8df18ff0202ce904180286a4a85
https://github.com/libgit2/rugged/blob/development/ext/rugged/rugged_commit.c

[0] http://stackoverflow.com/questions/12649697/how-to-commit-with-ruby-bindings-for-libgit2/12651234
