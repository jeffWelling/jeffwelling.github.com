---
layout: post
title: Fixing rack-protection
---

Today, I ran into a problem on my Debian development system. In coding something with a friend of mine, I needed to install Sinatra so that we could get a quick Web UI going to be able to both see the output at the same time.  I ran into a problem though.

		jeff@dev> sudo gem1.9.1 install sinatra ~/Projects
		ERROR: While executing gem ... (ArgumentError)
		invalid byte sequence in US-ASCII

Googling for this answer produced some unhelpful results, so I started to dig into this a bit further.  It seemed to be a locale problem, so I checked `env` but didn't find anything problematic. Regardless, I tried one of the suggestions of setting some environmental variables to specify a locale.  No change.  

So, I thought, if it's not my environment, it must be the gem.  So I forked the Sinatra gem on Github and cloned it to my local machine -- this would make it easier to submit a pull request later when I found and fixed the bug.  After cloning, I edited the Gemfile to contain one of the fixes I had found.  I built the gem and ran gem install again -- but again felt no love. After some coffee,  head-scratching,  and chin-hair stroking, I tried strace.

		open("/var/lib/gems/1.9.1/cache/rack-protection-1.5.0.gem", O_WRONLY|O_CREAT|O_TRUNC, 0666) = 6
		ioctl(6, SNDCTL_TMR_TIMEBASE or TCGETS, 0x7ffffa3631b0) = -1 ENOTTY (Inappropriate ioctl for device)
		write(6, "data.tar.gz\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 15872) = 15872
		close(6) = 0
		stat("/var/lib/gems/1.9.1/cache/rack-protection-1.5.0.gem", {st_mode=S_IFREG|0644, st_size=15872, ...}) = 0
		open("/var/lib/gems/1.9.1/cache/rack-protection-1.5.0.gem", O_RDONLY) = 6
		ioctl(6, SNDCTL_TMR_TIMEBASE or TCGETS, 0x7ffffa362e90) = -1 ENOTTY (Inappropriate ioctl for device)
		read(6, "data.tar.gz\0\0\0\0\0\0\0\0\0", 20) = 20
		close(6) = 0
		open("/var/lib/gems/1.9.1/cache/rack-protection-1.5.0.gem", O_RDONLY) = 6
		ioctl(6, SNDCTL_TMR_TIMEBASE or TCGETS, 0x7ffffa362e30) = -1 ENOTTY (Inappropriate ioctl for device)
		lseek(6, 0, SEEK_CUR) = 0
		read(6, "data.tar.gz\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192
		lseek(6, -7680, SEEK_CUR) = 512
		lseek(6, 0, SEEK_CUR) = 512
		lseek(6, 12054, SEEK_CUR) = 12566
		read(6, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 234) = 234
		read(6, "metadata.gz\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 3072
		lseek(6, -2560, SEEK_CUR) = 13312
		lseek(6, 0, SEEK_CUR) = 13312
		rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
		read(6, "\37\213\10\0R @Q\0\3\334W\333n\3436\20}\327W0OyhM;\t\212\26\2\272\260"..., 1137) = 1137
		rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
		close(6) = 0
		write(2, "ERROR: While executing gem ... "..., 85ERROR: While executing gem ... (ArgumentError)
		invalid byte sequence in US-ASCII) = 85
		write(2, "\n", 1
		) = 1
		rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
		rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
		getrlimit(RLIMIT_STACK, {rlim_cur=8192*1024, rlim_max=RLIM_INFINITY}) = 0
		futex(0x7f72f1d98a04, FUTEX_WAKE_OP_PRIVATE, 1, 1, 0x7f72f1d98a00, {FUTEX_OP_SET, 0, FUTEX_OP_CMP_GT, 1}) = 1
		futex(0x7f72f1d989c0, FUTEX_WAKE_PRIVATE, 1) = 1
		rt_sigaction(SIGINT, {SIG_IGN, [], SA_RESTORER|SA_SIGINFO, 0x7f72f179aff0}, {0x7f72f1a97780, [], SA_RESTORER|SA_SIGINFO, 0x7f72f179aff0}, 8) = 0
		rt_sigaction(SIGINT, {SIG_DFL, [], SA_RESTORER|SA_SIGINFO, 0x7f72f179aff0}, {SIG_IGN, [], SA_RESTORER|SA_SIGINFO, 0x7f72f179aff0}, 8) = 0
		close(5) = 0
		close(3) = 0
		close(4) = 0
		exit_group(1) = ?

Alright, now we're getting somewhere.  Looking at where it prints the error message, we can track back up the output to see what the previous operation was:


		read(6, "\37\213\10\0R @Q\0\3\334W\333n\3436\20}\327W0OyhM;\t\212\26\2\272\260"..., 1137) = 1137
		rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0
		close(6) = 0
		write(2, "ERROR: While executing gem ... "..., 85ERROR: While executing gem ... (ArgumentError)
		invalid byte sequence in US-ASCII) = 85
		write(2, "\n", 1
		) = 1

So, there was a read from file descriptor 6, and then it closed up and printed the error.  Ok, so what's file descriptor 6?

		open("/var/lib/gems/1.9.1/cache/rack-protection-1.5.0.gem", O_RDONLY) = 6

So, the problem may in fact lay with the rack-protection gem! The progress, we are making it and it tastes delicious! Alright, so let's fork and clone the rack-protection gem and try the same suggested fix to it's Gemfile. 1 rebuilt gem later,

		root@dev:/home/jeff/Projects/rack-protection# gem1.9.1 install -l rack-protection
		Successfully installed rack-protection-1.5.0
		1 gem installed
		Installing ri documentation for rack-protection-1.5.0...
		Building YARD (yri) index for rack-protection-1.5.0...
		Installing RDoc documentation for rack-protection-1.5.0...
		root@dev:/home/jeff/Projects/rack-protection# gem1.9.1 install sinatra                
		Successfully installed sinatra-1.4.2
		1 gem installed
		Installing ri documentation for sinatra-1.4.2...
		Building YARD (yri) index for sinatra-1.4.2...
		[warn]: Could not find readme file: README.rdoc
		Installing RDoc documentation for sinatra-1.4.2...

Winning!  Alrighty then! Let's push to Github and submit a Pull Request, then have cake! Mmm, cake.
