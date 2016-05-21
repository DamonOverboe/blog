---
title: "Ansible & Windows - Running a basic installer"
date: 2014-08-31
categories: devops ci continuous-delivery ansible windows
---

*In this post, I share my settings, playbooks and commands used to run
an installer on a remote Windows client.*

## Previous Articles

If you haven't yet, please read my previous posts on Windows + Ansible,
as this article expands on those. These will help you get a basic 
Ansible + Windows configuration set up, and should prevent some of the
issues I encountered.

[Ansible & Windows - Basic setup][basic]

[HTTP 500s, Timeouts, Stack Overflows, and Out of Memory errors][sos]

[basic]: http://damonoverboe.org/post/ansible-and-windows-basic-set-up
[sos]: http://damonoverboe.org/post/ansible-and-windows-timeouts-stack-overflows-out-of-memory

## Purpose

I'm using Ansible to help set up my Jenkins clients via the Swarm
plug-in; but the first step with these for me is to ensure Subversion
version 1.7 is installed on the machines, because the builds I'm
managing are stored in Subversion. 

While this isn't absolutely necessary, I have encountered a few 
occasions where I've needed to run a `svn cleanup.` While I could just
blast away the workspace with Jenkins, this is a bit more efficient.

Also, running a svn client newer than 1.7 causes issues with Jenkins,
because the plugin doesn't yet handle check-outs newer than that; so 
using a newer version on your client will prevent Jenkins from being
able to read the workspace, defeating the whole purpose of putting the
tool on there in the first place.

This conveniently makes a good tutorial too, because we will prove
what version of a tool is present, rather than just the latest
available.

## Defining the hosts

In my hosts file at `/etc/ansible/hosts`, I define my Jenkins clients.
*These are the machines responsible for doing the builds, while I treat
the master machine differently.*

		# /etc/ansible/hosts
		[jenkins-clients]
		clone1.damonoverboe.org
		clone2.damonoverboe.org
		clone3.damonoverboe.org
		clone4.damonoverboe.org
		clone5.damonoverboe.org
		clone6.damonoverboe.org
		
I also have `/etc/ansible/group_vars/jenkins-clients.yml` with the
appropriate settings:

		# /etc/ansible/group_vars/jenkins-clients.yml
		# ansible remoting
		ansible_ssh_user: dummy
		ansible_ssh_pass: dummypassword
		ansbile_ssh_port: 5986
		ansible_connection: winrm
		
And, because we're storing passwords on disk in the clear (one of the
many *benefits* of not being able to use SSH on Windows), you should
consider encrypting this file using 
[ansible-vault](http://docs.ansible.com/playbooks_vault.html).
It has a quirk or two but works fine once you learn how to use it; I'm
not using it here to keep this simple.

## About Playbooks

Playbooks are the sets of instructions that tell your clients what to 
do. Other deployment engines have recipes, directives, etc... playbooks
are Ansible's equivalent to that.

I store mine at `~/src/ansible-playbooks` and manage them with Git. I 
have subdirectories as needed to help further organize the files, 
including one for Jenkins. I would recommend doing something similar
to that.

## Basic playbook - stock `win_msi` module, no variables

The `win_msi` module is a bit limited, but is a good place to start
if you have an .msi file to install and can verify its installation
by checking a file path.

You'll probably need to use the `win_get_url` along with this to
download your file too.

Here is a basic example, not using any variables *yet*.


		# ~/src/ansible-playbooks/jenkins/clients-winmsi-setup.yml
		---
		- hosts: jenkins-clients
			tasks:

			- name: get svn client - version 1.7
				win_get_url:
					url: 'http://sourceforge.net/projects/tortoisesvn/files/1.7.15/Application/TortoiseSVN-1.7.15.25753-x64-svn-1.7.18.msi/download'
					dest: 'C:\TEMP\tortoise-1.7.msi'

			- name: ensure svn is version 1.7
				win_msi:
					path='C:\TEMP\tortoise-1.7.msi'
					creates='C:\Program Files\TortoiseSVN'
					state=present

You launch that playbook simply by running the following command:

		cd ~/src/ansible-playbooks
		ansible-playbook jenkins/clients-winmsi-setup.yml

					
## Trond's modified `win_msi` module

If you have Trond's changes to the `win_msi` module, you should be able
to add the following two lines to your `win_msi` task, providing a bit
more assurance you have the correct version:

		# enhanced win_msi module
			- name: ensure svn is version 1.7
				win_msi:
					path='C:\TEMP\tortoise-1.7.msi'
					creates='C:\Program Files\TortoiseSVN'
					state=present
		# add these lines:
					msiname='TortoiseSVN'
					msiversionstring='1.7.15.25753'

This is on the mainline `devel` branch on 
`git@github.com:trondhindenes/ansible.git`.

## Trond's `win_package` module - no variables

While the additions to the `win_msi` module are nice, the `win_package`
promises to deliver even more.

First and foremost, it can handle EXEs in addition to MSI files.

It also provides a way to pass command line arguments to your installer,
which is nice because this lets me tell the TortoiseSVN installer in
this case to include the command line client.

Also, you can generally bypass the `win_get_url` step, providing the
download url as the path variable for `win_package` instead. *Note that
currently, you are not able to rename the destination of the download,
and because Windows relies on file extensions rather than headers in the
binary files, this currently will not work with downloads that do not
have the extension on them, such as the TortoiseSVN link I'm using.*

So without further ado, here is the basic playbook using this:

		# ~/src/ansible-playbooks/jenkins/clients-package-setup.yml 
		---
		- hosts: jenkins-clients
			tasks:

			- name: get svn client - version 1.7
				win_get_url:
					url: 'http://sourceforge.net/projects/tortoisesvn/files/1.7.15/Application/TortoiseSVN-1.7.15.25753-x64-svn-1.7.18.msi/download'
					dest: 'C:\TEMP\tortoise-1.7.msi'

			- name: ensure svn is version 1.7
				win_package:
					path='C:\TEMP\tortoise-1.7.msi'
					name='TortoiseSVN 1.7.15.25753 (64 bit)'
					product_id='{6973F88F-DE2D-4DE9-81CF-98AA490FE417}'
					creates='C:\Program Files\TortoiseSVN'
					ensure=present
					arguments="ADDLOCAL=CLI"

## Let's clean it up with variables

Ansible provides the ability to use variables, stating that sometimes as
much as you wish it were true, not all systems are identical. This is
true if you're inheriting some other systems, and can be especially true
under Windows where there are not as many conventions as there are in
Linux *(some of the VMs I'm targeting have data partitions, some don't;
of the ones that do, some mount these at D: while others are E:; I can 
keep going...)*

Another nice thing about the variables are that they can make it easier
and safer to limit the clients you run this against. While there is the
`-l` option you can pass when calling a playbook, what if you forget to
pass it and you only wanted to test this on one client? It's probably
not the end of the world, but it's much easier to define your playbook
to require you to pass it.

In our playbooks, I'm making two changes.

1. **Change `- hosts: jenkins-clients` to `-hosts: '{{ target }}'`**  
This requires me to pass in the hosts via the target variable before
running, otherwise it has no clients to run against.

2. **Declaring the variable `root_path: 'C:\TEMP'`**  
This allows me to default the download path to `C:\TEMP` but override
it if I choose, in case I want to put it on the data partition. Note the
`vars:` section declared under the hosts.

Here is the modified playbook using `win_package`; changes to the others
are similar, and note the `dest` and `path` declarations for #2.


		# ~/src/ansible-playbooks/jenkins/clients-package-setup.yml 
		---
		- hosts: '{{ target }}'
			vars:
				root_path: 'C:\TEMP'
			tasks:

			- name: get svn client - version 1.7
				win_get_url:
					url: 'http://sourceforge.net/projects/tortoisesvn/files/1.7.15/Application/TortoiseSVN-1.7.15.25753-x64-svn-1.7.18.msi/download'
					dest: '{{ root_path }}\tortoise-1.7.msi'

			- name: ensure svn is version 1.7
				win_package:
					path='{{ root_path }}\tortoise-1.7.msi'
					name='TortoiseSVN 1.7.15.25753 (64 bit)'
					product_id='{6973F88F-DE2D-4DE9-81CF-98AA490FE417}'
					creates='C:\Program Files\TortoiseSVN'
					ensure=present
					arguments="ADDLOCAL=CLI"


To run this playbook, at a minimum I must provide a value for target.
To run against just one client, I can call:

		ansible-playbook jenkins/clients-package-setup.yml \
		--extra-vars 'target=clone1'
		
To run against all of the clients:

		ansible-playbook jenkins/clients-package-setup.yml \
		--extra-vars 'target=jenkins-clients'

And, to change where the download path is, I add that as a second
variable, overriding the value I've declared in the playbook:

		ansible-playbook jenkins/clients-package-setup.yml \
		--extra-vars 'target=jenkins-clients root_path=D:\downloads'
		
Best of all, if you fail to provide the target, it gracefully fails:

		ansible-playbook jenkins/clients-package-setup.yml
		
		> PLAY [{{ target }}] ***************************
		> skipping: no hosts matched
		> PLAY RECAP ***************************


## Learn more about the Windows modules

For the modules that have hit the mainline, visit the [Windows module][]
page.

But for newer modules that you may be testing out, you'll have to refer
to the files themselves.

For every Powershell script *(ending in .ps1)* at library/windows/, you
will find an accompanying usage file without an extension. Read that
file for more information on how to use the module; if you're running
against the development branch (which you pretty much need to do to get
the latest Windows features right now) this is generally going to be
more helpful than going to the [Windows module][] page.

If you want to try out Trond's `win_package` module, clone `git@github.com:trondhindenes/ansible.git` and check out the branch
`win_package`. You'll need the enhancements from the ansible 
[pull request 8759](https://github.com/ansible/ansible/pull/8759)
as well to make this run correctly.


[Windows module]: http://docs.ansible.com/list_of_windows_modules.html
