---
layout: post
title: "Escaping Darkness part 3...bragging time"
date: 2014-03-02
categories: ci devops scripting linux bash tools
---

[solo]: http://damonoverboe.org/post/meanwhile-six-months-later...
[second]: http://damonoverboe.org/post/escaping-darkness-part-2...the-back-story

## Agenda???

This is *almost* the last post in my series about the solo project and going dark.

* In [my first post][solo], I talked about the dreaded solo project, and set a rule to prevent this from happening again.

* In [my second post][second], I went a bit further on the same topic; what to do when it happens, and how to get out.

* Here, I'm going to brag a little bit...

* And in my future posts, I'll narrow the talk to some of the topics I find more interesting.

## The vacations ;-)

If I ask myself whether it was worth it to go dark, I have to look at my stats. The important ones, vacations:

* February 2013 - Labatt's Pond Hockey tournament at Eagle River, Wisconsin
* March 2013 - skiing @ Copper
* April 2013 - final game of the season with the Blues, meet & greet, Alumni box, etc.
* May 2013 - KCDC - mini vacation / geekdom
* June 2013 - Norfork Lake - family vacation
* September 2013 - vacation at Crested Butte; hiking, rafting, golfing, etc.
* September 2013 - Cards vs Cubs @ Wrigley; Cards won, Yadi hit a solo home run for the only run of the game, and I ran into my cousin Adam from St. Louis at a bar in Chicago!
* October 2013 - February 2014 - no vacations!!! *Boo this man*
* upcoming... March 2014 - skiing in Utah

So, I don't know. That last quarter and start of this year looks horrible to me. But, the ski trip is coming up soon, so that will help.

## Skills make it better???

If you throw in new or improved skills, along with some of the accomplishments, well, maybe:

+ Bash: I went from a beginner to a begginer that can drop your server; OK maybe a bit more than that, I'm probably intermediate now in my scripting ability. This will be one of my most enjoyable (and possibly least profitable) takeaways.
+ [sed](http://www.grymoire.com/unix/Sed.html): along with Bash, I grew a LOT here. I was between n00b and beginner before this, and I think I'm now at the intermediate level. This is probably even more enjoyable to me than Bash. I remember when I was first learning it, seeing people say things were so easy, just use this one line. Now, I can do that, and along with improved bashing skills, I can do it with confidence to entire directories of files. *I've been hacking against multiple 150k+ line files!!!*
+ [TeamCity][]: uggh. Powerful, clean as long as it's simple, clunky when you start trying to do too much. Try it out, and you'll see what I mean. It's so simple at first, then all of a sudden, it's a flaming pile of non-performing poo.
+ [BuildMaster][]: I wish I learned this tool before TeamCity, because I took some of the bad habits from TC and repeated them here; but, after cleanup, it's really simple and has a ton of helpers so you don't have to create some ridiculous deployment plan.
+ [SVN](http://svnbook.red-bean.com/): from former pro that fell to advanced over the past few years, back to pro. I didn't write the book, but I can now.
+ [Apache](http://apache.org) I still suck at it but I figured out a pattern and am hosting a few utils. My appreciation and knowledge grew a bit.
+ [IIS](http://www.youtube.com/watch?v=h-9UvrLyj3k&feature=kp): I still suck at it, I haven't figured out any patterns, command line is an afterthought for them, I hate Windows stupid `%variables%` and my hatred for both IIS and Windows has grown even more. You suck. It's not me because I don't get your products, it's you.
+ [Beer-drinking](http://www.beerknurd.com/stores/kansascity/): I'm still a pro, but I really kept those skills sharp.
+ [liquibase](http://liquibase.org) I'm just learning this one, but it seems to be pretty freaking cool and relatively easy to use so far. I'm 4 days in and it's going well.
+ [Hockey](http://www.imdb.com/title/tt0076723/): that's been my escape. I went from playing one day a week to three days a week; I've improved a lot but also enjoy the additional workouts.

## OK, Accomplishments...

And finally, the stuff my customer probably *(hopefully???)* cares about:

+ Migrated a ton of tools and added some new ones to a central data center:

	- two wikis, *again, boo this man!*
	- [Q2A](http://www.question2answer.org/) (open source StackOverflow),
	- all of the SVN libraries with massaging as needed for all of the product offerings
	- build automation - there are about 30 projects currently in [TeamCity][]
	- deployment automation - still in progress; 1 product is now almost fully automated in the deployment; the second is heavily assisted, and the third will be what I'm working on next
	- Changed the .Net stack from a running branch *(no trunk at all)* to a simple svn branching and tagging strategy: [unstable trunk/trunk-based development](http://www.ericsink.com/scm/scm_branches.html)
	- [Maven][] with [Artifactory][] and [TeamCity][]:
	
		- Maven is slick!!! And simple! I'm a .Net developer with about 5 minutes of Java experience, and I picked it up in absolutely no time. It's intuitive, convention based, mature and has easy to find and read documentation, and works absolutely flawlessly with continuous integration. For you NuGet nuts, you need to thank Maven; it's where NuGet is hopefully headed.
		
		- [TeamCity][] works very well with Maven *(because Maven simplifies builds and defines everything you need to know, so TC has nothing to do...)* and it publishes the build artifacts to Artifactory
		
		- [Artifactory][] handles receiving, storing, organizing, and serving up the Maven artifacts. In my not even remotely humble opinion, Java does a great job of managing different versions of libraries, allowing for multiple versions to sit in the same directory and being wise enough to know to take the latest *but you can use Maven to restrict the range of acceptable ones)*. Artifactory handles serving up the different versions.
		
[BuildMaster]: http://inedo.com/buildmaster/overview
[TeamCity]: http://www.jetbrains.com/teamcity/
[Maven]: http://maven.apache.org/
[Artifactory]: http://www.jfrog.com/home/v_artifactory_opensource_overview
