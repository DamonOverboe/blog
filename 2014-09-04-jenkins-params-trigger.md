Title: Jenkins Parameterized Trigger plugin
Date: 2014-09-04
Tags: devops, ci, jenkins, plugins

Based on the article [Orchestrate your delivery pipelines with Jenkins][orchestrate], I was testing out the [Parameterized Trigger plugin][plugin] on a new, clean project, because I was having issues when trying to add it to an existing project.


## Project Setup

I created two new projects:

1. **sandbox-trunk**  
A simple project to trigger my downstream pipeline

2. **sandbox-pipeline**  
A [Multijob][] project that I'll use in production to [orchestrate the pipeline][orchestrate]. This should be triggered by `sandbox-trunk`.

Both are basically empty. `sandbox-trunk` checks out a small repository, then I'm using the [parameterized trigger plugin][plugin] only to kick off the `sandbox-pipeline`.

In `sandbox-pipeline`, the only thing I did was add a shell script to echo out parameters that I expected would be passed to it. I modify those on occasion, but otherwise, all of the changes below are made to the upstream `sandbox-trunk` and am leaving the downstream `sandbox-pipeline` alone.

Ultimately, where I'm headed with this is there will be multiple feature branches that, after a successful build, will trigger the deployment pipeline. Each feature branch will have its own project, whereas the pipeline will be one set of projects that accept variables from the upstream feature branches.

		feature A   -\
		feature B	------>		deployment pipeline
		trunk		-/


## First Attempt - include *"Current Build Parameters"*

I had added "Current Build Parameters" which I figured would have been enough to trigger the build, but no. I ran several builds, but it never triggered the pipeline.

![img1][]

This might be a *"Thanks, Mr. Obvious"* statement, but when trying to trigger a downstream build with the [Parameterized Trigger plugin][plugin], it won't trigger the build unless you specify a parameter, and so my guess was that it wasn't including any paramaters.

This makes sense, since I hadn't defined any on this project, but I assumed it would pass the environment ones that it knew of.

## Second Attempt - add *Predefined Parameter*

So, I added a dummy parameter to see if that would trigger it:

![img2][]

And, it did. I wasn't checking in the pipeline for that `$face` parameter, but I *was* checking for the built-in `$SVN_REVISION` and it was empty, so I guess it's not included in the *Current Build Parameters*. This proved out my thought on my first attempt, that you must actually pass it a parameter. For further proof, I also found [this comment](https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin?focusedCommentId=38930765#comment-38930765) on the plugin page.

## Third Attempt - add *SVN Revision*

I left the dummy parameter in place from above, as well as the current build parameters, and added a third picking the pre-canned entry *Subversion Revision*. Since this is the first step in the chain, I am leaving the *Include/passthrough Upstream SVN Revisons* option unchecked.

![img3][]

And the result? Nope... it won't pass the SVN Revision. So I turned on passthrough, and same result. Take a look at the parameters link on my downstream `sandbox-pipeline`:

![img4][]

That's not surprising though, the comments section of this plugin is littered with comments that it doesn't actually pass the revision, and there are workarounds for that.

## Fourth Attempt

**Add a parameter to the project, keep *Current Build Parameters*, remove everything else.**

This is to test to see if it was just simply that I'm not including any parameters in the upstream project. Near the top of the upstream project, I added a choice parameter:

![img5][]

... why not?

And then in the post-build, reverting back to just passing the *Current Build Parameters* from the first attempt:

![img1][]

And the result of that, the downstream `sandbox-pipeline` reports is actually triggered and has the variable:

![img6][]

## One last quirk

From the Linux world, I learned that all-caps parameters should be reserved for global ones to the system, and I should use lower-case for mine. It's just a convention, but it does help you prevent overwriting the system variables in your session.

For example:

		echo $path
		#
		echo $PATH
		# /a/whole/bunch:/of/stuff/here
		
		path="Linux is case-aware"
		echo $path
		# Linux is case-aware
		echo $PATH
		# /a/whole/bunch:/of/stuff/here
		
I had noticed that a lot of examples in Jenkins, people use all caps for their variables as well, but I decided to stick with my convention of using lower case for my variables:

![img7][]

And while I know shell scripts on Windows probably aren't *"fully"* supported, they do support it and even have documentation telling you to set the location of `sh` for your Windows environments. *This works well with UnxUtils or MSysGit, and they mention Cygwin of course.*

Anyway, it seems that lower-case variables in the Windows shell aren't supported; they come through just fine, but the shell doesn't catch them. And to test whether Jenkins (or a plugin) is converting them to upper, I put both lowercase and caps in the downstream project.

My shell script *in the downsnteam `sandbox-pipeline`*:

![img8][]

The parameters that it received:
![img9][]

And its console output::
![img10][]

So while they're passed in with the correct casing, it does seem that when they're passed to the shell, they're converted by Jenkins.

And as to be expected, Windows doesn't care about your case:

![img11][]
![img12][]


*Tested on Windows Server 2012, [Jenkins][] 1.578, [Parameterized trigger plugin][plugin] 2.25*


[img1]: ./images/param-trigger/img1.png
[img2]: ./images/param-trigger/img2.png
[img3]: ./images/param-trigger/img3.png
[img4]: ./images/param-trigger/img4.png
[img5]: ./images/param-trigger/img5.png
[img6]: ./images/param-trigger/img6.png
[img7]: ./images/param-trigger/img7.png
[img8]: ./images/param-trigger/img8.png
[img9]: ./images/param-trigger/img9.png
[img10]: ./images/param-trigger/img10.png
[img11]: ./images/param-trigger/img11.png
[img12]: ./images/param-trigger/img12.png

[orchestrate]: http://www.infoq.com/articles/orch-pipelines-jenkins

[Jenkins]: http://jenkins-ci.org/
[multijob]: https://wiki.jenkins-ci.org/display/JENKINS/Multijob+Plugin
[plugin]: https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin
