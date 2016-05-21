---
title: "Implementing Continuous Integration with TeamCity for Multiple Branches with Multiple Promotions - Part 0"
date: 2013-06-11
Tags: teamcity, ci, continuous-integration, c#, .net

*A journey through implementing Continuous Integration across multiple branches using TeamCity v. 7.1.4.* 

+ Part 0 addresses the current situation and the plan going forward.
+ [Part 1] starts breaking the build steps up into stages of promotion, covering the build + unit tests.
+ Part 2 will continue with breaking up the build steps, working on the integration tests and setting up for the automated UI tests, deploying for Manual Exploratory Testing if all succeeds. 
+ Part 3 will address extracting this template and using it to manage all branches.

## The problem at hand

A current client of mine uses what I will call a *"running branch"* model of branching in Subversion. Basically, nothing gets merged back down to trunk, and all development happens out of a branch named after the upcoming release. When it's time for the next release, the current release is essentially treated as the trunk, and a branch is created off of it for the new revision. 

While this is less than ideal and definitely not the model I like to use, it really doesn't present too many challenges over what you would find with a shop that had trunk-based development and a couple release candidates or feature branches. It does result however in having to change your configurations pretty frequently. It also is not your normal approach to branching, so it adds a bit of confusion since it deviates from the norm.

Basically, at any given time there are at least two branches and maybe up to five that we would want to watch for changes. One huge benefit is that they have an aggressive updating model and do not allow their customers to get behind on versions, which will eliminate needing to watch too many branches.

## The Plan

Currently, we have TeamCity running against the active branch only, however a new branch has been spun off, so it's time to update. Additionally, all of the build steps are contained in one configuration.

I am starting with [these steps and guidelines,][basicRecipe] but then breaking up the full build and deploy promotion process into this:

![image: Build Promotions](https://dl.dropboxusercontent.com/u/16078906/teamCity/1.process.png)

So, I will be adding support for a new branch, while breaking the build steps into the build chain shown above. This:

+ makes the promotions / failures at each stage more visible;

+ provides quick feedback on whether a unit test has failed;

+ if unit tests pass, it promotes the artifacts for integration testing (which takes considerably longer than the unit tests);

+ and if integration passes, it allows us to specify a particular build agent for the automated UI tests.

If the idea of build artifact promotion is new to you, I recommend [this article](http://thedailywtf.com/Articles/Release-Management-Done-Right.aspx) by [Alex Papadimoulis](http://thedailywtf.com/Authors/Alex_Papadimoulis.aspx) for a quick but very informative read.

## Getting Started - Preparing for Step 1.

If you do not already have automated builds on Team City working, start with the [above example][basicRecipe] and get a check-in build working for your source repository. This will get you through any of the kinks or gotchas and you'll have automated builds at least on your check-in.

If you do not currently tag your successful releases, or maybe need to collaborate with others or get permission before auto-tagging, you can skip the point where it auto-tags on success. It would be a nice to have though, but you will also see that we will be creating a set of artifacts as we go forward to test against; we will build only once, so the tag really only exists to help you see in the SCM which revisions successfully passed all tests.


**[Next in Series: Part 1][Part 1]**


[fixme]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[basicRecipe]: http://www.troyhunt.com/2010/11/you-deploying-it-wrong-teamcity_25.html
[autoTag]: http://www.laurentkempe.com/post/Build-and-Deployment-automation-VCS-Root-and-Labeling-in-TeamCity.aspx
[part 0]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-0
[part 1]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-1
[part 2]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-2
[part 3]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-3

