---  
layout: post  

title: My boot problem
---  

I've had a problem with one of my servers for quite a while now.  Luckily, it was one of those intermittent problems that only reared its head long enough to be a pain in the ass, but not quite a big enough pain as to make me spend more than an hours worth of time troubleshooting it.  And being a hardware problem, it can often take a bit longer then that to troubleshoot.

I have a server with several hard drives plugged in, most via SATA but one or two via USB. I installed Debian and happily configured my system.  Then, one day, I had a drive die on me.  Never mind the ugly pain it was trying to identify which drive failed, that's a story for another time, we'll just skip to the part where I have figured out which drive has failed. After turning off the machine and unplugging the drive, I turned it back on again and expected a successful boot. I was watching the screen boot, I saw the regular BIOS output and memory tests, and then I saw the blinking cursor that [experience told me] indicated that it couldn't find grub.

I was perplexed, I had just done a base install, how could it possibly have a boot problem now, I know that the root partition was not on the drive that had failed. The drive that failed was part of a RAID array and the root partition was not even on the RAID array. I hadn't even done anything like reformat any drives so it seemed quite unlikely that I had accidentally blown away my root drive.

Confused, but not knowing where to begin troubleshooting, and needing a boot-able system rather quickly, I felt I had no choice but to just reinstall the operating system and restore the files on the RAID from backup. Had this been anything other then a personal side-project at home, I would have felt an obligation to determine the cause of the problem before just reinstalling the operating system because without knowing how it happened in the first place you cannot know for sure that it won't happen again. I reinstalled the operating system, and tried some basic tests like plugging in a USB drive and rebooting the system and it seemed stable. Luckily, when I went to build a new array out of the old drives, I found that the system had already rebuilt the array based on the filesystem type on each of the drives. Before adding the drives to the RAID I used fdisk to set the "partition's system ID" to "Linux raid autodetect".

    fdisk /dev/sda
    Command (m for help): t
    Hex code (type L to list codes): fd
    Command (m for help): q

This it seems allowed mdadm to rebuild the array at boot without the need of a configuration file (I had never set the configuration file for mdadm at all on any system).

Some time later, I bought a replacement drive for the RAID array to increase the stability.  I use RAID6 and so the array can sustain a loss of up to two drives without any loss of the files on it. Lose 3 drives however, and your files vanish in a puff of bytes. At that point if memory serves I was running the array while being down 2 drives, so if I had another drive failure before I added a new drive, my files were gone, so I put together some funds and got a replacement.  Well, after installing the replacement drive and trying to boot the system, I ran into the same problem of the system being unboot-able.

I wasn't sure how to troubleshoot a problem like this, and personally I loathe hardware problems because of how inconsistent they can be and how hard they can be to track down with certainty, so for a long time I ignored the problem by A) having the good luck of not needing to replace any more dead drives, and B) by not rebooting the system very often if at all and making sure the same number of drives were plugged in as when I last booted.

A while later I moved to a new place with a geek room mate and so of course I had to make sure my computers worked and were in pristine condition. Some people might not understand the sublet competition that can take place when two geeks live under the same roof, it's like a subtle little geeky arms race, and when one of those people is a Mac advocate and one is a hardcore FOSS advocate, that competition can shall we say intensify a bit. So maybe you now understand why I had to make sure that my systems were in expected working condition.

One night while having some long-time friends over to visit who also happened to be infected with the geek virus, got bored and so we decided to take a go at troubleshooting why this computer had a problem booting. We turned it off, rebooted it, turned it off again, unplugged a drive, rebooted it, tried unplugging another drive, and so on. Eventually we came to the conclusion that one of the ports on the SATA card had a problem with it because whenever a drive was plugged in to that port the system wouldn't boot.  

So, happy as can be that we solved the problem, I configured the system and got it ready for "production" here at the house. Then, I had another drive die in the RAID array, which isn't unexpected because by now these drives are a couple years old at least as is the rest of the computer. So I turned the machine off and unplugged the drive, but when I rebooted I was again greeted with the blinking cursor of doooooom. 

At this point I was getting exasperated by this decidedly irritating problem, and so resolved to go with a solution that I knew worked -- PXE boot the system. Since the problem appeared to be that it couldn't even find grub let alone the root partition, this felt like it should work. I rebooted the system into the BIOS and went rooting around for the configuration options to set the system to boot from the network, and while I was in the system I noted that the drive that was listed first among a list of the available drives that were plugged in was a Hitachi drive, and I only had one of those and it was plugged into the SATA card (as opposed to a slot on the motherboard itself). This clued me in to what could be the problem, when a drive was plugged in to the SATA card the system thought of it as coming before the drives that were plugged into the SATA slots on the motherboard. 

So, having an idea what the problem was now, I had an idea for a solution.  I would install grub to each of the drives attached to the system via SATA. So I did, and so far everything has been working perfectly.  I even tested by going through another bout of unplugging and replugging a bunch of various drives and it booted every time.  

TLDR -- The problem seems to be that because I was using a SATA card, drives attached to the SATA card were considered to be before the drives plugged into the SATA ports on the motherboard, and so the system had installed grub to one of the drives attached to the SATA card.  When that drive died, the master boot record on it went as well. By installing grub to each drive the system is booting consistently, even when I add a new drive or remove an old one. I just have to make sure I install grub to each new drive installed, that way it doesn't matter what drive the system tries to boot from.
