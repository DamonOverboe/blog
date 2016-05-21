---
title: "Ansible & Windows - Basic set up"
date: 2014-08-25
Tags: devops, ci, continuous-delivery, ansible, windows

*Updated on 2014-08-30*

I started this a month or so ago, and had relatively few issues. At the
time, the PowerShell script for adding a Windows client was just a pull
request, so there was a bit more manual hacking to do on your own.

Since using the [helper script][] has been integrated into the
[mainline of the documentation][doc], much of the preamble and the
helper steps have been removed, making your life a lot easier in
setting up a remote Windows client.

*Note that you must still have a Linux control machine, and there
are no plans to ever change that. That's a good thing, because
this tool was originally created for controlling Linux machines and
has had a lot of development and testing.*

Start by following along with the [Windows Documentation][doc], 
completing the steps they list there; below are a few snags you
might hit while completing that, but at the end, you should be
able to control your remote Windows hosts.


## Snag 0 - launch PowerShell as an Admin

Because, adding the ability to elevate privileges on the command line
*(ahem, `sudo`?)* would make too much sense... so instead, bypass the 
security completely for the entire session by right-clicking on 
PowerShell and choosing `Run as Admin.`

I should warn that you're now an admin on Windows, but if you have done
anything with Windows, you already know how terrible this whole process
is. Onto snag #1.

## Snag 1 - enable PowerShell scripts on the client

If you're going to run the PowerShell script, well, they have to be 
enabled. 

If you're like me and know next to nothing about PowerShell and find 
Windows help to be absolutely atrocious *(how about a `usage` section at
the top?!?)*, you'll be staring at a big red pile of error puke, 
telling you that you cannot run a script.

The error will look something like this gigantic TL;DR pile of poo. 
I've cut the paths out to make this a bit more legible???:


<pre><code>		.\ConfigureRemotingForAnsible.ps1
		
<font color="990000">		# .\ConfigureRemotingForAnsible.ps1 : File 
		# .\ConfigureRemotingForAnsible.ps1 cannot be loaded
		# because running scripts is disabled on this system. For 
		# more information, see about_Execution_Policies at
		# http://go.microsoft.com/fwlink/?LinkID=135170.
		# At line:1 char:1
		# + .\ConfigureRemotingForAnsible.ps1
		# + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
		#	+ CategoryInfo : SecurityError: (:) [], PSSecurityException
		#		+ FullyQualifiedErrorId : UnauthorizedAccess
</font></code></pre>

If you care, you can check what the scope is by running this:

		Get-ExecutionPolicy -Scope CurrentUser
		
		# Undefined
		
**To fix this problem, run the following:**
		
		
		Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

		# Execution Policy Change
		# The execution policy helps protect you from scripts that 
		# you do not trust. Changing the execution policy might expose
		# you to the security risks described in the 
		# about_Execution_Policies help topic at
		# http://go.microsoft.com/fwlink/?LinkID=135170. Do you want 
		# to change the execution policy?
		# [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
		
Answer **Y** or just press enter since it's the default. Re-run 
your script and all should be good. Just an FYI, you have turned 
on remote scripting on your Windows host. It was off by default,
so I'm going to go ahead and venture you've just opened yourself 
up to all kinds of pwning attacks, but you get what you pay for 
*(and you or someone you work for or represent paid for Windows, 
congrats!)*

## Snag #2 - Domain related: Authentication fails

**You need a local administrator account to connect to on the remote
machine.**

The [Windows documentation][doc] mentions that you need to be
logged on as the admisitrator, but it's easy to skim over it.

If you're familiar with Ansible already, then you're coming from
the Linux world and would hopefully know better than to allow
remote commands like this to run as the `root` user. And if you've
worked in any enterprizey Windows shop or have an ounce of common
sense (or have been hacked / virused up / etc) then you should also
know you don't want to run as the main machine Administrator.

Unfortunately, you'll still need an administrator account to run
this, so you'll still need to take your security pretty seriously.

And currently, there is not support for Active Directory,
so you must have a local administrator account. That's good and bad,
but we can worry about that later when we have the choice of 
which way to go. For now, create a local account and add them to 
the Administrators group.

*NOTE there is currently a fork that is adding AD support; it's
being code reviewed and tweaked as needed, and I suspect it will
eventually make it to the mainline, but I'm not following the status
of it so that's just a guess.*

## Snag #3 - Errno 110 - Connection timed out

If you're connecting to a computer on the same domain as your 
host and you immediately try to ping using the ansible module 
`win_ping` *(assuming you've  done all of the other steps; host 
file, passwords, etc)* you'll probably hit a firewall error.

The reason for this is because the firewall rule in the script is 
set to Public. *(My guess is because on the Linux side, it's 
cheaper and easier to just get a bunch of disposable VMs from 
Digital Ocean or similar)*

So, on your target client, change the firewall rule to allow for 
Domain connections.

**You can correct this by doing one of the following:**

1. **Easiest: modify the helper script before running it.**  
Look for the following entry, and change `Profile=public` to be 
`Profile=Domain` or `Profile=Any`, depending on your needs:  
  
		netsh advfirewall firewall add rule Profile=public name="Allow WinRM HTTPS" dir=in localport=5986 protocol=TCP action=allow
  
2. **Win UI: use the Windows Firewall** mangler... same as above on the
scope.

3. **Run the PowerShell script only.**  
Take the entry from #1, modify it appropriately and run it in PS.

## Snag #4 - 'FAILED => winrm is not installed'

*updated 2014-08-30*

This is an id10t problem, due to me skimming the documentation.

The steps are simple:

1. Set up a Linux control machine, following the main
[installation](http://docs.ansible.com/intro_installation.html)
instructions.

2. Run one additional step for Windows, as seen in the [Windows version of Installing the Control machine](http://docs.ansible.com/intro_windows.html#installing-on-the-control-machine)

		sudo pip install -U http://github.com/diyan/pywinrm/archive/master.zip#egg=pywinrm

If you care to see all the fun exploratory stuff I encountered, you can 
go [here](https://groups.google.com/forum/#!topic/ansible-project/Qhg2N7OSBbY).

I had flown through setting up the Control machine and assumed I could
bypass the Control Machine setup section on the Windows page. Oops.


## Snag #5 - Random Stack Overflows, Out of Memory, and HTTP 500s

*added 2014-08-30*

These are most likely caused by a Windows bug. Read more about it
[here](http://damonoverboe.org/post/ansible-and-windows-timeouts-stack-overflows-out-of-memory).


## Testing it out

Afterward, run the `win_ping` module again. If you have an inventory
file listing:

		[windows]
		winsvr1.example.com
		winsvr2.example.com


Then you can run the following to ping all of them:

		ansible windows -m win_ping
		
Or this to just ping server 1:

		ansible winsvr1* -m win_ping
		
And you should see something like this:


<pre><code><font color="006600">    winsvr1.example.com | success &gt;&gt; {
        "changed": false, 
        "ping": "pong"
    }
</font></code></pre>
		

These are just the basics. Pinging doesn't do much for us, but,
it proves we can connect to the remote machines. Currently, there
are not a lot of [Windows modules][lib]. But, you can deploy
and run remote scripts, as well as issue some raw commands, and I 
expect the [Windows module library][lib] to grow quickly.

I will definitely be exploring this more, as it might actually make
deploying code to a Windows host a little less painful than 
having kidney stones.



[helper script]: https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1

[doc]: http://docs.ansible.com/intro_windows.html

[lib]: http://docs.ansible.com/list_of_windows_modules.html
