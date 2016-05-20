Title: Unclear Error Messages in TeamCity
Date: 2013-10-15
Tags: svn, ci, java, devops, teamcity, maven

For the most part, the error messages generated by TeamCity are very clear and tell you exactly what has failed.

However in the past week, I have ran across two messages that would lead me to think one thing, when in reality, one was most likely a bug and the other due to using the program other than they intended.

## Error deploying artifact (which was previously succeeding)

I had a project that had previously been deploying to Artifactory successfully, and then all of a sudden started failing. I looked at the change logs, and there wasn't a change that had actually caused it to fail either. This project was a dependency of another project, and a manual build in that project triggered this one.

The help message itself isn't very, well, helpful. Looking at the verbose log, it says that Artifactory denied the artifact.

One would immediately think (or at least I did) that is permission based, however I have others properly deploying to it.

Digging into the Artifactory logs, I see that it's trying to create a directory, however, the jars actually exist and are valid.

The best I can tell, this was due to corruption of the build configs when updating from 8.0.2 to 8.0.4, because:

1. I manually replicated  the build config; it worked.

2. I copied the failing build config into a different project; it worked.

3. I copied the failing build config into the same project; it worked.

For more history and details, see [my post on StackOverflow](http://stackoverflow.com/questions/19260932/teamcity-fails-deploying-to-artifactory-reports-expected-a-folder-but-found-a). If you suddenly see a project that was previously succeeding to deploy and then for no apparant reason fails, and then 



## Patch is broken, can be found in file

Also coupled with this was the error:

> Problem with Maven Snapshot Dependencies Trigger

This was interesting. I have a build template for the Maven projects. Each project resides in its own repository, so we trick the VCS root repo by setting it to:

	http://svn.myCompany.com/%repoName%/

This wasn't something I came up with, but rather something I repeated because it seemed to work. No doubt this probably causes some confusion to TeamCity, depending on when / how it resolves the repoName variable.

In addition, for the checkout rules I use `%pathInRepo%` to determine wheter it's trunk, branches/abc, etc.


How I ran across this problem was:

1. I set up a build config in TeamCity and pointed to it's original location.

2. The path in the repository then changed, such as branch abc was copied to branch def, and then abc was dropped later.

3. I changed the TeamCity build parameter `%pathInRepo%` to point to the branch def.

4. The patch error occurred, and also, the Maven pom error showed up, claiming it couldn't read the POM.

I suspected this is due to server-side checkouts and not being able to patch since this was now a different branch, despite it being in the same repository. I tried forcing a clean, etc, but in the end I had to delete the project and create a new one.

The POM error is just annoying, but I think that stems from the way I'm abusing the repository. It's coming from the build trigger based on Maven dependency changes. This branch is actually a branch off of trunk, with no changes to the pom. In fact, they even have the same sha1 sum!

The TeamCity build config covering trunk is happy with no warnings about the pom, so I think this one is that TeamCity didn't expect me to use this the way I am (no real root repository, create a branch, then change the branch). I'm going to let cleanup run overnight and see if it still complains.

The way to get around it, temporarily, is:

1. Changed build configs to point to trunk

2. Clicked on the build project, went to the Maven tab, and hit the "Reread Maven Project Settings" button.

3. Changed the settings to point back to branches/abc.

I know this is a false sense of security, but I don't care for the time being. There is no reason TeamCity cannot read the POM other than it's own failure.

As a proof, I created a new build config in the sandbox, and attached it to a new VCS root, which I pointed at the actual VCS root (replacing the `%repoName%` with a value) and then kept the checkout rules as branches/abc. This resolved the Maven pom correctly, and ran correctly.

If the overnight cleanup doesn't resolve this, I guess I'll submit a bug on this too.

*Note: This appears to have resolved itself overnight. If I see this error occur again in the future, I'm probably going to have to create unique VCS roots for each repo.*



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
