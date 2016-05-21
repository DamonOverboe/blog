---
title: "Migrating TWiki"
date: 2013-09-19
categories: howto linux tools fml
---
Published: true

**a/k/a How I Help Keep Mother's Brewery in Business!**

Here we go. I expect a lot of heavy drinking, gray hairs, overall frustration and a bit of hatred of life. This thing was simple to install and serve up, but a friggin beast to even get it to accept a user login!

## Backing up the install

Because, well, I don't want to fight with getting TWiki to let me create a user again:

		tar -czf ~/backups/twiki/webroot.twiki.gz /webroot/twiki
		tar -czf ~/backups/twiki/etc.apache2.gz /etc/apache2

## Pull down the data from the Source Server

This resides on a box named `wiki`; in case you're sarcastically challenged, I changed the host name; this isn't actually hosted on `networkWithPileOfCrapTwikiSoftare.com`. Well, at least I don't think it is. I generated a new key pair, sent the public key over to the admin, and then edited my `~/.ssh/config` file to add the necessary credentials for the host first *(see [this link][multiSSH] for more info)*:

		Host 1.2.3.4
			User			iHateTwiki
			Hostname		wiki.networkWithPileOfCrapTwikiSoftware.com
			IdentityFile 	~/.ssh/pileofcrapsoftware/id_rsa
			PreferredAuthentications 	publickey

Unfortunately though, in TWiki's cumbersome, verbose fashion, you have to read about 20 pages of documentation. I *LOVE* tools whose upgrade path is *"Read [this massive PILE](http://ow.ly/p27Zx) of doc."*. So much better than intuitive tools that just do it for you or have an obvious upgrade path!!! Thanks TWiki, you're the best!!!

<!--
		ssh 1.2.3.4
		
-->






[img1]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img2]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img3]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img4]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img5]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img6]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img7]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img8]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img9]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img10]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img11]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img12]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img13]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img14]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img15]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img16]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img17]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img18]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img19]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[img20]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg

[multiSSH]: http://www.robotgoblin.co.uk/blog/2012/07/24/managing-multiple-ssh-keys/
