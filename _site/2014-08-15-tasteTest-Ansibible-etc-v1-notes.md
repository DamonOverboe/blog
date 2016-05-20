Title: Taste Test - Ansible, Salt, Puppet, Chef
Date: 2014-08-15
Tags: devops, automation, tools, continuous-delivery

## Intro

About a month ago, I took the
[taste-test][] by Matt Jaynes, comparing Ansible, Salt, 
Puppet, and Chef. I had some very basic experience with Puppet and 
familiarity with Chef, but hadn't really delved into any of the tools.

*I had purchased the book about 6 months ago, so I was using version 1. I
just noticed that he has updated the book, so the workarounds below may
no longer apply. The version I had was working against Ubuntu 12.04 LTS,
whereas I ran against 14.04 and the latest versions of each of the 
tools.*

I highly recommend the book, as it repeats the same scenario with SSH
plus 4 different tools yet takes the pain out of learning the tools.

You'll see that he states its not for people working in the Windows
stack, but I disagree. The book does assume you have a background with 
Linux and some familiarity with SSH, but that's probably not required.
You'll have the commands you need to run, and you can always look up the
topics or commands to get a better understanding. And, I know if you're 
reading this post, you probably know me, so just reach out to me
whenever you hit any snags and I can guide you through it. *I guess if
you don't know me, thanks for stopping by!!! Go ahead and reach out
anyway if you hit snags.*

## &*<@(*&#$ RANT!!!!!!!!!! and the reason for taking this taste test.

My motivation was, well simply, I'm absolutely sick of the notion of
having to write your own deployment tool to deploy code in the Windows
stack. I have some experience with BuildMaster and knew about Puppet and
Chef, but I wanted more exposure.

I've worked in the Windows stack my entire career *(and my liver
hates me for it)* and every company I've worked with has some convoluted
process to ship code. It's a problem that plagues Windows development,
and it started back when they decided the mouse and dumbass UI was their
key to dominating the market.

Most efforts are some cobbled together process that involves expensive,
terrible and outdated tools, coupled together with some scripting glue,
all of which are so fragile everyone is afraid to touch them. The amount
of manual effort I've seen it take to ship code on the Windows stack are
absolute bullshit and extremely error-prone. I've never been one to be
accused of praising Microsoft, so this rant probably comes as no
surprise, but I'm just absolutely tired of it and all the hoops you have
to jump through just to ship some stupid 1s and 0s.

Admittedly, not all of the blame falls on Microsoft... maybe only 90% of
the fault is theirs. OK It's probably more like 51%, but why not shift 
the blame to what their market share is, because until recently they
were strictly tight-lipped and closed source company. Anyway, all of us
stupid .Net developers perpetuate the problem by not forcing any kind of
conventions on our processes, and putting up with the absolute crap they
force on us.

I could go on ranting for days, and probably still only touch the 
surface of my anger against this stack, but our failure to impart any 
kind of standards makes our lives difficult. In turn, that guarantees 
that you must have some technical idiot to maintain the chaos; they'll 
probably make the problem even worse and make your process even more
fragile, and the stupid pattern just continues. I'm absolutely sick of 
it.

Rant over, I hope. Enter, Continuous Delivery tools, and then we can
hopefully figure out how to bend them to work with Windows and impart
some conventions on how we act. *I'm talking about so much more than
just putting your application at some ridiculously long file path with
spaces and parentheses and a processor identifier...*


## Overall

Again, this is based on version 1 of [the book][]. This section should
help with all of the tools, but then you'll get into a section for each
later on.

	+ There are a few tasks to do on all machines that will help.
	
		- **add keychain** *(for saving ssh keys)*
		- **edit /etc/hosts** on each, adding master in puppy & kitty, and 
		puppy & kitty on master
		- **symlink for nginx**:  
		This book uses Ubuntu 12.04, and the nginx version used in 
		this book references `/usr/share/nginx/www/` as the web root.
		However, using the latest nginx 1.4.6 & Ubuntu 14.04, the web root is 
		`/usr/share/nginx/html`. This needs to be adjusted for in all of the
		instruction here, *OR*, as you're setting up your clean environments,
		create the target directory and symlink to it:
		
				mkdir -p /usr/share/nginx/html
				ln -s /usr/share/nginx/html /usr/share/nginx/www
	
	+ If I give access to others, I should either remove the password from
	the ssh files or set up alternate paths in the .ssh/config
	
			

## Chef

	+ Page 66: adding bootstrappers to puppy and kitty
	
		- the puppy.dev and kitty.dev need to be listed in the 
		/etc/hosts file on the work.dev machine; it doesn't take this from
		the master.dev machine (I tested it)
	
		- also, it uses the ssh keys from work.dev, not master.dev; odd. But
		if you use a different private ssh key on the work.dev than the
		master.dev (which I did after I created it) then you'll need to 
		register it's key on the puppy.dev and kitty.dev machines as well.
		
		- and, it looks like the clones need to have /etc/hosts entries to
		master as well. joy. It makes it through a lot and then throws a
		network error later.
		
## Puppet

	+ Page 72: the referenced debian package is old and missing. 
	Names / versions can be searched for [here](http://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html)
	but if you know exactly what you need, just find the correct package
	from the [apt page](https://apt.puppetlabs.com/). 14.04 is Trusty.
	
	+ Page 73: when launching the clients, a warning is thrown but doesn't
	seem to prevent execution:
	> Warning: Setting templatedir is deprecated. See 
	> http://links.puppetlabs.com/env-settings-deprecations
	> (at /usr/lib/ruby/vendor_ruby/puppet/settings.rb:1095:in 
	> \`block in issue_deprecations')
	
	To resolve this, simply comment out that line in the `[main]` section
	of `/etc/puppet/puppet.conf`: 
			`# templatedir=$confdir/templates # deprecated`
			
## Closing

If you want to skip reading the full book, just do Ansible. It probably
wouldn't hurt to know Chef and Puppet as well, as they're very popular,
but that's just handy if you go somewhere else that already has this
established.

OH! And my D.O. bill, $0.25! I was pretty diligent about destroying
the servers when I wasn't using them. That's a bit wasteful to save a
few pennies but there were a few gaps in between me being able to 
complete this and I didn't want to pay for them sitting idle.

Then, adding the promo code UBUNTUDROPLET, I received a $10.00 credit! 

Onto Ansible + Windows!

[taste-test]: https://devopsu.com/books/taste-test-puppet-chef-salt-stack-ansible.html
[the book]: https://devopsu.com/books/taste-test-puppet-chef-salt-stack-ansible.html