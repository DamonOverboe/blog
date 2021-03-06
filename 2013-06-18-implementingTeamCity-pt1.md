Title: Implementing Continuous Integration with TeamCity for Multiple Branches with Multiple Promotions - Part 1
Date: 2013-06-18
Tags: teamcity, ci, continuous-integration, c#, .net

*A journey through implementing Continuous Integration across multiple branches using TeamCity v. 7.1.4.* 

+ [Part 0][] addresses the current situation and the plan going forward.
+ Part 1 starts breaking the build steps up into stages of promotion, covering the build + unit tests.
+ Part 2 will continue with breaking up the build steps, working on the integration tests and setting up for the automated UI tests, deploying for Manual Exploratory Testing if all succeeds. 
+ Part 3 will address extracting this template and using it to manage all branches.

## Recap

In Part 0, we outlined a plan for build promotions upon successful tests, and if you did not already have basic builds happening on check-in, then you should now have those set up and working.

Here we will begin introducing promotion steps, starting with Build + Unit Tests and deploying the artifacts. We will use those artifacts in Part 2 to run the Integration tests against.

## The Differences

In the [article I referenced][basicRecipe] in Step 0 the author simply uses TeamCity to create the build, implies some of what you should do as intermediate steps, and if all of your intermediate steps succeed, he then leverages TeamCity to handle the promotion to deployment-ready in Subversion, auto-tagging the trunk at the given revision. *This was detailed in an article linked in the [above article][basicRecipe], [here][autoTag]*. That is extremely handy and I intend to do the same, however I'm going to provide some body / thinking to the intermediate steps.

If you do not have a build configuration yet, the the [article I referenced][basicRecipe] is your starting point. Implement it first, and then come back here to modify it or build a new configuration.


### Specifying the Artifact Paths for Build Chains

In order to have the build chains available, we must tell TeamCity what to deploy, and where to deploy them to. Here you simply go to `General Settings : Artifact Paths` and specify the binaries you want to bundle, and package them up to use later:

![image: Edit artifact paths](https://dl.dropboxusercontent.com/u/16078906/teamCity/artifactPaths.png)


### Using MSBuild instead

One of the first deviations I have run into from the basic implementation is that in the [referenced article][basicRecipe], the author uses Visual Studio as the build step. Instead, I am targeting MSBuild and specifying the .Net framework to use. Your mileage may vary, but this gets around several build problems that we were facing.


### Adding Unit Test Coverage

Next, the author mentions that you can implement the test runs here. I am going to go head and do that. I should mention that we have mostly separated unit tests from integration tests, however we have multiple projects for each, so we have to build a configuration file for xUnit. If you do not have that scenario, it should be a bit easier for you to set up xUnit.

While still at `3. Build Step`, which now says `Build Step: MS Build` for me, I edit those settings, making the following changes:

1. Change the name to Build & Unit Test

2. Add a new Build Step to build the xUnit project file *(optional, ymmv)*

	a. Choose Command Line as the runner type.
	
	b. Name it "Build xUnit project file for Unit Tests".
	
	c. Change Run to "Executable with parameters".
	
	d. In Command executable, target our batch file that builds the project file *(it simply matches our unit test project files naming convention and creates a config file)*.
	
	e. In Command parameters, I specify `Tests.Unit` which simply tells our batch file that we want to find all projects under that folder.
	
	f. Then after saving, now is a good time to click `Run` to make sure it works. Your build step 2 should look something like this:  
	![image: Build Step 2 - build xUnit project file](https://dl.dropboxusercontent.com/u/16078906/teamCity/buildStep2.png)
	
3. Add a new Build Step for running the unit tests with Code Coverage

	a. Choose ".Net Process Runner" for the runner type.
	
	b. Name the step "Run xUnit Unit Tests".
	
	c. Set the Path to point to your xunit.console runner from your source. *We added a folder titled `InternalDependencies` for DLLs that TeamCity or our development environments depend on but that the product does not, and these are not shipped to the customer.*
	
	d. If you have a xUnit config file, as we do from step 2 above, then pass it as a command line parameter; ours is simply `Tests.Unit.xunit`
	
	e. Set the correct .Net runtime and platform.
	
	f. With Version 7.1 of TeamCity, code coverage is easy to enable, simply change the .Net Coverage tool to "JetBrains dotCover"
	
	g. The rest of the settings should be able to be left as is, however we follow the convention of making sure all of our tests have `Tests` as the root namespace, so I excluded our tests from the coverage by setting an Assembly Filter of `-:Tests.*`.
	
	h. Save and run; you build step three should look something like:  
	![image: Build Step 3 - Run xUnit unit tests](https://dl.dropboxusercontent.com/u/16078906/teamCity/buildStep3.png)  
	and after a successful run, you should see a successful run and code coverage under your Project's overview page.  
	![image: Project page with code coverage][https://dl.dropboxusercontent.com/u/16078906/teamCity/buildSteps-CodeCoverage.png]

When finished, you should have three Build Steps, and it should look something like this:

![image: Build Steps overview](https://dl.dropboxusercontent.com/u/16078906/teamCity/buildStepsAfter.png)

## Conclusion

At this point, we have our CI configured for automated builds on check-in. Because we left each step as the default *"Proceed only if previous succeeds"* a failure in building or a unit test failure will cause this build to be considered broken.

If that succeeds, then artifacts are deployed to our specified location for more automated testing down the road.

In the next part, we'll create a build chain, picking up those build artifacts and run the Integration Tests against them.


**[Previous in Series: Part 0][Part 0]**
**[Next in Series: Part 2][Part 2]**



[fixme]: /home/damon/Dropbox/Photos/graphics/clipart/constructionDuck.jpg
[basicRecipe]: http://www.troyhunt.com/2010/11/you-deploying-it-wrong-teamcity_25.html
[autoTag]: http://www.laurentkempe.com/post/Build-and-Deployment-automation-VCS-Root-and-Labeling-in-TeamCity.aspx
[part 0]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-0
[part 1]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-0
[part 2]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-0
[part 3]:http://damonoverboe.org/post/implementing-continuous-integration-with-teamcity-for-multiple-branches-with-multiple-promotions-part-0
