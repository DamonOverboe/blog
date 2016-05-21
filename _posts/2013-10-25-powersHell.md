---
layout: post
title: "PowerShell needs your network drives!"
date: 2013-10-25
categories: error scripting powershell rant
---


For those of you who have been putting off or avoiding learning PowersHell altogether, I think you made the right choice.

They made some nice improvements over the standard *"don't call it DOS"* Windows commands, but they made some really silly omissions too.

One of the most annoying to me is they made a new terminal for it, and yet, copy & paste are still just as horrible there as they are in the old command window. Only now, you don't have to click "mark", it just automatically goes into marking mode.

But here's a gem. I restarted a VM, and trying to run scripts after the reboot (which were working fine before), I get a nice and ugly red error:

	Attempting to perform the InitializeDefaultDrives operation on the 'FileSystem' provider failed.
	
What's the real problem? Well, either you have ["something like SoftGrid installed"](http://social.technet.microsoft.com/Forums/windowsserver/en-US/d2747041-d650-42c6-89bd-495d1c8d4578/powershell-launch-error-after-upgrade) or you mapped a network drive and it's been disconnected.

I understand that if a network drive isn't available, then obviously, you won't be able to do anything against *it*, but why would you make it so that your entire scripting engine doesn't work? Bizarre.
