## Overall recommendations

1. Use BuildMaster now for Meridian deployments.

2. Consider moving 100% into BuildMaster, rather than TC+BM or Jen+BM; license fees would be:

	- Up to 20 users: $2,995

	- Up to 35 users:	$4,995
	
	- Up to 50 users: $6,495
	
3. Or, keep memr in BuildMaster for now, but begin transitioning everyone off of TeamCity.

	- Rather than getting Joe up to speed in BuildMaster, work with him to call their scripts from Jenkins
	
	- That trains him in the tool, and really, the tool could be managed by any and all developers,
	
	- But you want your technical leads to help oversee the workflow and keep it simple.
	

Regardless, from what I've seen, I would recommend transitioning away from TeamCity.


## BuildMaster & Liquibase


+ BuildMaster is extremely close to ready. 

	- It could be used now for standing up QA environments,
	
	- although you know the memr developers will complain about it "not using their scripts"
	
+ Currently, those are sitting still. I haven’t heard anything from Joe or Chuck. 

	- **Do I need to just reach out to them and book some of their time?**
	

## Jenkins vs TeamCity

+ I have the build chains working in Jenkins.

	- I have the uro portal build as API, Portal (same build step, no chain)
	
	- I have memr utils_library => client build chain set up.
	
	- I have the main urochart application building; I haven't set up downstream projects like faxing, etc but that would be next.
	
+ I have the feature branches working with all of the above

	- They are extremely easy to create, (creating in Jenkins makes the branch & clones the build project)
	
	- and extremely easy to reintegrate.

	- I used this with Portal last week (working with Justin, Patrick & Greg) to create feature branches, and that’s working well; I had Justin do the work, essentially demonstrating it to him.

+ You can have an unlimited number of masters and clients with Jenkins

+ It is the tool of choice by a ton of development communities;

	- so, it would be easy to find paid support, employees or contractors familiar with the tool.
	
	- and there is a ton of help online.

+ Permissions are very easy in it; 

	- it uses Active Directory for authentication (and auto-detects; it was a breeze, whereas working with Mario, we couldn't get TeamCity to communicate with the AD)

	- and you can grant / deny rights to existing groups (such as `healthtronics/ci_admins, qa, developers` ...)
	
	- so people automatically get the rights they should have; no monkeying with users is needed,

	- and you can also create local groups for permissions (but the domain groups would be cleaner)

+ I don’t see that it offers any better or any worse integration with BuildMaster than TeamCity offers.

+ But it could possibly replace the need for BuildMaster…

	- It does have the ability to do promotions (with permissions restrictions)

	- The BuildMaster workflow and UI is better suited to QA / Testers

		+ But the Meridian stack is heavily developer-driven,

		+ so they could set it up to automatically stand up new environments upon successful tests
