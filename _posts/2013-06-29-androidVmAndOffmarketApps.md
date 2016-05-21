---
layout: post
title: "Setting up an Android VM - and adding 'incompatible' apps"
date: 2013-06-29
categories: Android VM Howto VirtualBox
---

The following are the steps to set up an Android Virtual Machine on your computer using VirtualBox. 

However, you'll find that a lot of the software available in the Play store may not be compatible with your new install, which will report as a VirtualBox tablet or something like that, so we'll also work around that.

## Start the downloads on your PC

1. If you do not already have it, [download VirtualBox](https://www.virtualbox.org/wiki/Downloads) by Oracle. Get the Extension Pack as well, there should be a universal binary for the version you downloaded on the main Download page.

2. While that is downloading, read the instructions [from TheHowtoGeek](http://www.howtogeek.com/164570/how-to-install-android-in-virtualbox/) for setting up and Android VM, and start the download of the `.iso` image from the site mentioned there.
*If that link dies, Google "Howto Geek Android VirtualBox" and you should get there.*

3. If you have a slow connection, you can jump to the work on your phone if you anticipate certain apps will not transfer, then come back here. Otherwise just continue.

4. After you have completed the above, launch the Android VM. If you cannot see your mouse pointer, on the VM's menu go to `Machine: Disable Mouse Integration`.

5. While the system is booting, it will prompt you to set up wireless. You DO NOT need to do this, because your internet connection from your host will be passed to the VM guest. Just hit skip.

6. After the system boots, go to the Google Play store and start installing your basic apps.

7. I won't go into it in depth here but when you have your basic apps, I recommend taking a snapshot in VirtualBox. This is essentially a backup copy of the entire VM.

## Start the work on your phone

If all the apps you want are in Google Play, you're stylin'! But, I'm going to bet that they're not. So now you simply need to get them onto your tablet.

1. Add AndroZip to your phone. This does not require rooted phones.

2. Inside of AndroZip, use their "Apps" button in the upper right to display all applications on your phone

3. Select the apps you want to migrate to your VM and choose `Backup`. This will copy their `.apk` files to an `app_backups` folder at the home level.

4. Get these apps into your VM somehow. I used Dropbox, but if they are small enough, you can email them. There are other options, but if you do not know about them (and maybe even if you do), then I would say these are the easiest two.

## Installing to the VM

Once you have the files into your VM, you need to adjust some settings.

1. Add AndroZip to your Android VM guest. You'll be installing the applications "from disk."

2. Go into your main settings on the Android VM, then click Applications, then clear the check box that prevents loading from untrusted sources. If you can't find it, go on to #3 and it will bring you back to the right spot here.

3. Browse to the desired `.apk` file using AndroZip and click it. This should start the install process.

4. If #3 fails for any reason other than the security setting, then you probably will not be able to run the app in the VM.
