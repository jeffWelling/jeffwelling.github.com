---
layout: post
title: Documentary Review of The War On Kids
---

I just ran across my first bug when developing my first Android app!

I'm following along with an EBook called Android Apps For Absolute Beginners, and I got to Chapter 7 when I ran across the first bug in my code. In all previous instances of potentially problematic code, the IDE had called me out on it, but in this instance the IDE was telling me that everything was fine.  However, when I ran my app and clicked on an item in the menu, the app crashed.

I first tried attaching the debugger and going at it from that angle, but that proved fruitless after mucking around and not being able to find any helpful indications of what was going wrong.  My next approach was to read the source code scouring for something, I didn't even know what, just anything that stood out as wrong. Several days later, I threw a couple more hours at the problem and ended up using breakpoints to figure out exactly where in the code the crash was occurring.

The crash was occurring on this line:

              ImageView image = (ImageView) findViewById(R.id.ImageView01);

It turned out, the object I was referencing with `R.id.ImageView01` was a TextView, and was the wrong object.  After changing the reference to point to the proper object, the app loaded and works as expected.

It just seems like since the IDE doesn't warn of this problem, it's worth noting it in case I come across it again in the future.  Coming from a weakly typed Ruby background, these kinds of type related problems aren't as familiar to me as they may be to someone coming from a strongly typed language.
