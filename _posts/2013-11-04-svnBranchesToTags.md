---
title: "Two simple Bash commands to tag & drop many svn branches"
date: 2013-11-04
categories: svn scripting linux bash
---

I'm doing a bit of cleanup of some old branches in Subversion for a current client of mine. These were all released but never tagged; each release had its own long-lived branch (almost immortal, until today).

I broke it into two steps so I could test it on a small set first, but the following commands:

1. list the branches that match my pattern into a file
2. then tag and drop each branch listed in that file


Note I did not put the branch/tag names in quotes; these branches do have spaces but I did not end up needing to escape them. YMMV. Also, I have no doubt there are more efficient ways to do this *( and I'd love to hear them: [+damon.overboe](https://plus.google.com/u/0/105021269922813532736/posts/p/pub) or [@DamonOverboe](https://twitter.com/DamonOverboe) )*, but it took me more time to write these notes so I can repeat this in the future than it did to run this.

Also note **I am running these directly against the repository!**


0. Prep - store your IFS value; we'll set it to null later to preserve whitespace:
	
	OLDIFS=$IFS

1. Next, set up the pattern and dump your branches; make sure your pattern goes through to the `/`

	pattern="Your Grep Pattern Here.*/"
	svn ls ^/branches | grep -e $pattern | tee branches.log

2. Read the branches dump, tag and drop
	
	while IFS= read -r line; do echo $line; svn cp ^/branches/$line ^/tags/$line -m"tagging branch $line" && svn rm ^/branches/$line -m"removing the branch $line; refer to the tag instead."; done < branches.log

3. Cleanup - Restore your IFS
	
	IFS=$OLDIFS




If you have nested branches under a placeholder, you'll need to modify those a bit. I addded another variable `branchHolder` for that purpose, and worked it in on the `svn ls $branchHolder` and then into the read command:


	while IFS= read -r line; do repo="$branchHolder/$line"; svn cp $repo ^/tags/$line -m"tagging branch $repo as tags/$line" && svn rm $repo -m"removing the branch $repo; refer to the tag tags/$line instead."; done < branches.log





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

