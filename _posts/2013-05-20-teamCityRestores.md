---
title: "Restoring Team City to a new (or clean) environment"
date: 2013-05-20
Tags: continuous-integration, howto, linux, windows

I started with the directions [here](http://confluence.jetbrains.com/display/TCD7/Manual+Backup+and+Restore), which discuss manually backing up and restoring an environment in Team City. Below I have highlighted the differences or struggles I encountered while following these directions.

My suggestion is to open that documentation, and keep this open along side it. You may want to skim the headers here first to see what kind of problems you may run into.

## Getting Started

Most of the information was accurate, but I ran into a couple issues with it when restoring. Most of these were there in the information, but my first time through, they weren't very clear.

On my first run-through, I restored into the embedded file database for proof that it would work.

### TeamCity must NOT be running ###

They say this, but also tell you to start it at a different spot, and you will probably need to start it at least once to get the database templates. Just make sure you stop the service if you started it.

### Java must be installed and set up ###

with the environment variables for `JAVA_HOME` and appended to `PATH`

- I used:

		nano ~/.bash_profile
		
		# ... editor ...
		
		export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-i386
		export PATH=$PATH:/usr/lib/jvm/java-7-openjdk-i386/bin
		
		# ... <c>+o to save, <c>+x to exit ...

- see [this post][javaHomePath] for more info or background

- Note that I had to drop `/bin/java` from the `JAVA_HOME` variable; You'll know whether you need to do this if Team City says something like *".../bin/java/bin/java could not be found."*


### Make sure your data location has plenty of free space ###

The main Team City install with one agent took about 4gb, and then there was another 4gb of data that I was restoring. 

By default, this folder is named `.BuildServer`, and will be in the installing user's home or user directory.

- The easiest way to avoid this is to set the `TEAMCITY_DATA_PATH` environment variable before starting.

- If you install under Linux, you probably sudo'd, so this puts it under your root user's home directory. My root's home is on a small partition.

- If you are using partitions, you may run into this.

- If you run into this you will need to either:
	
	+ Grow the partition
	
	+ Add a system environment variable to point elsewhere (same for all OS now, you want to add an environment variable `TEAMCITY_DATA_PATH` similar to how you added Java above)
	
	+ Move the folder and then point to it with a symlink

### The data directory *(.BuildServer)* folder needs to be empty, BUT ###

In my scenario, I wanted to restore from MySQL into the File Based database because all I wanted to do was test out the restore and set up code coverage.
	
- Since I wasn't going to the same schema / database, I had to specify my target Database.

- The file-based database exists at `.BuildServer/config/database.hsqldb.properties.dist`, **however remember the BuildServer file has to be empty?!?**

- While the documentation isn't clear on this because it states Target, these are actually only config properties that you're going to use to set up your database.

- So specifying `-T .BuildServer/config/database.hsqldb.properties.dist` fails, because you have to delete that file,

	+ Therefore **MOVE** that file out of the .BuildServer root first, and then target it.
	
	+ I ran:

			mv .BuildServer ./BuildServerOld
			
			bin/maintainDB.sh restore -F <full file name of TeamCity backup file> -T ./BuildServerOld/config/database.hsqldb.properties.dist

	+ Maybe that's common sense to you, but it seemed odd that I was specifying a different "Target" than where it was actually going to install to and that I must have that file, but I wasn't allowed to have the file in the configs folder.

- I have confirmed that the referenced file in `-T`, that while they call it a target, it most definitely **IS NOT!!!** the Target database. Instead, it contains the *TEMPLATE* for the Target Config Settings, and the word Target is misleading in my opinion. Knowing that it is a template for a database would help, and TEMPLATE would be a very important word that would have made it a bit more obvious or common sense that you would need a copy outside of the config folder as a starting point. Maybe I'm just dense, but that's not what the documentation said to me. Anyway, here's the proof if you need it:

	+ I checked the last accessed time, and then moved the file in BuildServerOld:

			cd /root/home/BuildServerOld/config
		
			ls -lu database.hsqldb.properties.dist
		
			sudo mv database.hsqldb.properties.dist database.hsqldb.properties.dist.old

	+ looked at the timestamp at the new install directory:

			cd ~/.BuildServer/config
		
			ls -lu database.hsqldb.properties.dist

	+ Then restarted the service:

			sudo /usr/bin/TeamCity/config/bin/runAll.sh start

	+ Simply the fact it started was probably good enough, but to verify, I rechecked both of the timestamps above. Sure enough, the one in ~ had been accessed, the one on root had not.


## Restoring into MySQL

This assumes that no database is set up on the machine where you want to install.

See [this link](http://confluence.jetbrains.com/display/TCD7/Setting+up+an+External+Database) for more info in regards to setting upn an external database for Team City. 

And [this link](http://stackoverflow.com/questions/3246482/mysql-command-line-client-for-windows) for accessing the MySQL console.
 

1. Open the MySQL console (or do it through the GUI I guess); I'm using my name *(damon)* as an admin example, substitute with root or whatever your admin account may be.

	- Windows:
	
			cd c:/progra~2/mysql/mysql*/
		
			mysql -u damon -p
		
	- Linux: 
	
		- you should not need to change directories, you should be able run from any location in the terminal unless your bash config is hosed.
		
		- simply run the `mysql -u damon -p` command

2. create the database and grant privileges; I'm again using my name in the examples, use whatever names you need, replacing the *damon* stuff:

		CREATE DATABASE damonTest DEFAULT CHARSET utf8;

		GRANT ALL PRIVILEGES ON damonTest.* TO damon@localhost IDENTIFIED BY 'thisIsNotReallyMyPassword';

3. copy the `database.mysql.properties.dist` from the data folder `.BuildServer/config` folder to another location outside of that location, and rename it `database.properties`. *NOTE to the earlier topic this makes it clear that this is just a stupid config template, and I still want to know why they didn't include them elsewhere!*

	- Linux:
	
			cp .BuildServer/config/database.mysql.properties.dist ~/database.properties
		
	- Windows
	
			copy .BuildServer/config/database.mysql.properties.dist c:/users/damon/documents/database.properties

4. Edit that file and set up your MySQL server, username and password:

		connectionUrl=jdbc:mysql://localhost:3306/damonTest
	
		connectionProperties.user=damon
	
		connectionProperties.password=thisIsNotReallyMyPassword

5. On Linux, I didn't have any issues, the necessary jars pulled down with the tarball. But on Windows, these are not bundled in the installer. So, you will likely run into a error stating something about not being able to find the mysql jar in `c:/programdata/jetbrains/teamcity/lib/jdbc`. If you downloaded the tarball instead, you should be fine. If not, you'll need to download that from MySQL. *I set up my Linux install first, and then went with the Windows install. So, I simply copied the jar out of my `.BuildServer/lib/jdbc` folder and dropped it under the ProgramData folder.*


## Finishing the Restore - applies to all Databases

1. Continue with the steps above, running these commands from the root of the installation

		`bin/maintainDB.sh restore -F <full path & name of TeamCity backup file> -T ~/database.properties` 
		
	- substitute `maintainDB.cmd` for Windows, using the same parameters 

2. Cross your fingers! Then go to the bathroom, get coffee, etc after you see it hit *"Importing data..."*  This goes pretty quickly when targeting MySQL, but is *SLOW* if targeting the embedded file-based.

3. Start the service back up, same as above, by running

	- Linux: 
	
			`bin/runAll.sh start` 
			
	- Windows:
	
			`bin/runAll.cmd start`

## After the fact - Changing the Port Number

On my first VM installation, I hit cancel to stop the installation but it only canceled that step?!?! The installation continued, and so I had to modify the port number afterward.

Then in the second environment, while I was letting the install happen on my host (I don't have a Windows VM set up on this machine), I was doing other work on my other environment. I somehow managed to get the popup to appear in my other VM, and hit enter by accident.   

So, in both cases, I had to go back and change the port number afterward. To change the server's port, in the `<TeamCity Home>/conf/server.xml` file, change the port number in the HTTP/1.1 connector (here the port number is 8111):

		<Connector port="8111" protocol="HTTP/1.1"
			connectionTimeout="20000"
			redirectPort="8443"
			enableLookup="false"
			useBodyEncodingForURI="true"
		/>

See [this link](http://confluence.jetbrains.com/display/TCD7/Installing+and+Configuring+the+TeamCity+Server#InstallingandConfiguringtheTeamCityServer-ChangingServerPort) for more information.

[javaHomePath]: http://www.cyberciti.biz/faq/linux-unix-set-java_home-path-variable/
