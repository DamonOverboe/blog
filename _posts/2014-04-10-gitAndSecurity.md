---
layout: post
title: "Host a private git repo + some cool defense tools"
date: 2014-04-10
categories: git scm howto linux tools security lvm
---

I have a few git repositories here and there that I access as file-based repositories; some put on network shares, others put in Dropbox, and I decided it was time to upgrade that.

Now, you're probably thinking Github or Bitbucket, but, you also know I like to complicate things a bit, and if I didn't, then you wouldn't have this rambling article to read! *Also, I needed to keep this in house on this project.*

## Tutorial article I used

For the server I'm using to host, I already have ssh set up, along with `.ssh/config` on all of my clients to make it easy to hit the host.

I found a few posts here and there, but for some reason or another, they weren't quite what I needed. Then I ran across [this question](http://askubuntu.com/questions/416254/how-to-setup-a-private-git-repository?rq=1) on Ask Ubuntu, which described my needs. Funny, it actually described the problem I was having too, I just didn't know it.

In that question is one gem of an article that the question's author used as a guide: [ted felix git server](http://tedfelix.com/software/git-server.html).

The tutorial is laid very out well, explaining the steps without going into too much detail *(something I always feel I struggle with)*.

He approaches the article as if you've downloaded Ubuntu LAMP, and you want to host git, so now what? So, it's a great start-to-finish tutorial, but if you are already partially there, it will definitely help you fill in the gaps.


## Defensive tools & hardening

As a very nice bonus, he covers some hardening techniques throughout the article, and lists some cool tools at the end. 

There were two tools that I should have known about already: `tripwire` and `denyhosts`, and both of these were available through `apt-get` on Debian distros.

Denyhosts simply blocks hosts that flood attack you, and tripwire sets traps around; not honeypots, but it does watch sensitive files for intrusion attempts. I have a lot of reading to do on it still.

## Things I did differently

### I do use LVM

If you've never used LVM before and you're just trying to set up git, and maybe you're not that familiar with Linux, you can consider skipping LVM for now. But, I highly recommend you come back to it, soon!

LVM stands for *"Logical Volume Manager"*. Have you ever struggled to resize your stupid c: partition on Windows that has the e: partition right behind it? Switch to Linux and use LVM, it fixes that!

There are two main reasons I use LVM:

1. **It is extremely easy to add more disk space, grow or shrink partitions.** AND, you can do it on a live file system, you don't even need to shut down your machine!

2. **Security!** When I first started exploring Linux, I ran across a [great article](http://www.linuxsa.org.au/tips/disk-partitioning.html) that discussed what partitions to use, what options to set, and why. LVM doesn't add security, but, the flexibility and ease of partitioning that LVM provides makes it easy to create and maintain the partitions you need. The quote:

> Next time you see...
>
>		[lame@nothought /]$ df
>		Fileystem      1k-blocks      Used Available Use% Mounted on
>		/dev/hda1       17654736   1354220  15403692   8% /
>
> ...you can do something about it!

### I kept SSH at the conventional port 22

I specifically did not change SSH from port 22 because all that changing that port does is add "security by obscurity." That could slow down a total n00b for a few seconds, but really, it just doesn't work. Whether in code or implementation, all that it ever really does is frustrate the crap out of you or someone else when they try to come in after you and figure out what the hell you were doing!

Want proof? On Linux, point `nmap` at your target server (or a range of servers) and see what ports are open, and what they report as. For other options, search Google for Amap and Unicornscan and see if you can find a download link for either of these tools, which are used to fingerprint open ports and help identify apps running behind them. Then consider that I'm not a hacker; I have had less than one week of formal white-hat training, and I'll read only a handful or two of security articles a year, yet I know about these tools.

_(**But, be careful!!** I do not vouch for the authenticity nor security of these tools; downloading or using them may increase your vulnerability.)_

Anyone that wants onto your system will have these or something better. Maybe if you leave it at port 22, they'll send you a thank you card later. So here, I choose to follow the convention of port 22.


### User Setup

To create a new user and give them rw permissions, there are a few mundane steps. Ted laid these out and explained them very well, using *username* to identify where you were supposed to place the user's name.

I simply substituted in `$username` and threw it in a script. It will add the user, set them to only allow ssh login, and add them to the git group so they have commit privileges.

I need to add some hardening around the user *(name already used, invalid chars, different key name/location and maybe use `find` instead to locate the user's keys)*, but here's a starter:

		#!/bin/bash
		
		# git-add-user.sh
		#
		# adapted from <http://tedfelix.com/software/git-server.html>
		#
		# creates a new user, changes to ssh only, adds them to git
		# this script requires su privileges, so it should be set to 
		# 744 / rwxr--r--:
		#
		# 	chmod 744 /usr/local/bin/git-add-user.sh
		
		echo "enter the user's name:"
		read username
		if [ "$username" = "" ]; then
			echo "invalid user name; exiting"
			exit 1
			
		elif [ ! -f "$username/id_rsa.pub" ]; then
			# I expect that we have already received the user's public key,
			# and that you have put it in a subfolder $username/id_rsa.pub
			
			echo "public key not found; giving up"
			exit 1
		fi
		
		# everything else is stock from Ted's article, using $username now...
		
		# add a user
		adduser --disabled-password --shell /usr/bin/git-shell $username
		
		# every user I add can commit, so add them to git
		gpasswd -a $username git

		# See Ted's note on needing a git-shell-commands directory
		mkdir /home/$username/git-shell-commands
		
		# add their public key to their authorized_keys file
    	mkdir /home/$username/.ssh
    	sh -c "cat $username/id_rsa.pub >>/home/$username/.ssh/authorized_keys"

    	# give them ownership on their directory
    	chown -R $username:$username /home/$username
    	
I'm also using this to help pull code onto multiple VMs, where I only need readonly access. So, I followed this article and created a generic user, `gituser,` and did not add them to the git group. This gives them read-only access of /srv/git. So, to bring on a new VM, I still generate a new pair of ssh keys, but I add them to `gituser`'s account's authorized\_keys.  

### That's about it

Other than that, I pretty much followed the article. I already had a LAMP server that I had locked down to SSH access only. One of the earlier posts I had read got me most of the way there on configuring git, but, the part that I missed that should have been completely obvious is I didn't create the git group, nor did I set the ownership on /srv/git to myself or any user group that I belonged to. 

For what it's worth, the error I was seeing really had me looking in the wrong places; it kept saying "host not found" so I assumed I had something configured incorrectly with Git because I knew I could shell into the server, but I never tested the /srv/git folder. 


