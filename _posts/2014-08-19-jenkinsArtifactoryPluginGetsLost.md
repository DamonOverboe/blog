Title: Jenkins Artifactory plug-in gets lost with mixed repositories
Date: 2014-08-19
Tags: devops, ci, continuous-delivery, jenkins, artifactory, maven


I've been exploring moving to Jenkins from TeamCity for both Java and .Net stacks for several reasons (but I'll summarize those at a later time).

So far, I absolutely love Jenkins, but it's taken a bit longer to get everything wired up the first time just due to the learning curve. However from what I've seen, when you get it working, it offers a *TON* that TeamCity does not.

## The situation

I added a *"Mavenized"* Java project to Jenkins, which produces a library used later / downstream by other Java projects. It has no outside dependencies, so it was extremely easy to configure and get successful builds coming out of it.

Next, I added a downstream project. I set it up to be triggered when the upstream built, *(super easy)* and I added the Artifactory plug-in because this project has a ton of dependencies, both internal and external.

The Artifactory plug-in lets you test the connection and will build a list of the available repositories for you to select, one for snapshots and one for releases. That was easy to wire up *(use http://path-to-artifactory-server/**artifactory**/)* and get the list coming back.

So I thought everything was going to be smooth sailing, but, *FAIL!* Over and over and over again, it was failing to resolve dependencies from Artifactory. It turned out that I wasn't reading the log close enough to see the true problem.

## The details

Artifactory supports virtual repositories, which proxy multiple repositories. This keeps your settings file small and generally makes life easy. Out of the box, you have `repo`, which is a virtual repository for `libs-release` and `libs-snapshot`. `libs-snapshot` is a virtual repository for `libs-snapshot-local`, `ext-snapshot-local`, and `plugins-snapshot-local.` The same pattern is used for `libs-release`, although it includes the global repositories too.

I kept hacking with Jenkins + Maven + Artifactory, and it kept choking, failing to resolve a snapshot dependency. I verified it was where it needed to be, and that the snapshots were set to pull from `libs-snapshots` but the error message kept saying it couldn't find it in `libs-release`, and I finally noticed that... for some reason, it was trying to pull the snapshot from the release repository.

So, what it came down to was that the `libs-snapshots` virtual repository had a couple older repositories that had been imported from Nexus, and these had mixed items, including both snapshots and releases.

The interesting thing is, and what finally helped me slow down and read the logs, is that it was reporting back an actual version with the timestamp, rather than just 5.6.1-SNAPSHOT. So, it seems that the first query to Artifactory by the plug-in properly tells the build exactly which snapshot it should be looking for, but a subsequent call to fetch the artifact gets confused if there are mixed repositories and causes it to look in the `libs-release` repository instead. I can make guesses as to why that happened, but as long as I know how to prevent it, I guess I don't care.


## The fix

To fix this and get the project to build, I:

1. Created a new virtual repository, named `libs-snapshot-only`
2. Added `libs-snapshot-local`, `ext-snapshot-local`, and `plugins-snapshot-local` to `libs-snapshot-only`, leaving the legacy ones out.
3. Changed the Artifactory plugin in Jenkins to use the newly created `libs-snapshot-only` virtual repo for the snapshots.

This allowed me to leave `libs-snapshot` as is for the time being, so it won't hose the developers (or TeamCity for that matter). And, as a result, it *FINALLY* built!


## Summary

So in summary, make sure your virtual snapshot repository does not include any repositories that allow for releases. Setting up Artifactory for a clean state defaults to this, however if you import from an older repository, those might not have been set up that way.


*Observed on Windows Server 2012, Jenkins 1.575, Maven Integration plugin 2.6, Jenkins Artifactory plugin 2.2.3, and Artifactory 3.0.1-r30008.*