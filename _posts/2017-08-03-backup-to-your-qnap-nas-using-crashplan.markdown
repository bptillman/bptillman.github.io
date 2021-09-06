---
layout: post
title: Backup to your QNAP NAS using CrashPlan
date: '2017-08-03 17:43:27'
tags:
- backup
- crashplan
- nas
---

If you already utilize backup software like CrashPlan/BackBlaze/TimeMachine to keep copies of your important files on your computer, then good on you. If not, you should get on that so that [Scott Hanselman won't have to keep asking you to do it](http://www.hanselman.com/blog/TheComputerBackupRuleOfThree.aspx). Like Scott reiterates in that post, it's not really enough for you to just back up to a separate drive on your computer but instead to backup that to an external source as well. This way you don't have a single point of failure.

For me, that external source right now is my QNAP TS-212 NAS, which worked really well as a TimeMachine backup for my Macbook, but was a bit more involved out of the box to get it to backup things on my Windows machine. It wanted me to install its own custom QSync software and set up its own backup/sync process, completely separate from my existing CrashPlan setup that was backing up to my internal backup drive. CrashPlan also had no way of being able to find my QNAP NAS over the network to try backing up that way via a network drive or UNC path either. All in all it didn't sound like there was a good solution.

This is where QNAP having the iSCSI feature came to the rescue for me. With it I was able to continue using my existing CrashPlan backup routine, but just have a second source for it to backup all my precious data to.

So let's get started shall we.

#### Enable iSCSI ####
First things first, in the NAS software open up the Control Panel, then find Storage Manager, and then click the iSCSI tab. In here you will need to check the first box to enable iSCSI, which should look something like this:
![](/content/images/2016/Aug/01-iscsi.PNG)
<br/>
#### Create the iSCSI Target ####
Next we need to create an iSCSI target, so we do that by clicking on Target Management link on the left in the screenshot above, and click the Quick Configuration Wizard to get to creating the iSCSI Target. In here you will need to give it a name and alias on the first page like so:
![](/content/images/2016/Aug/02-target.PNG)
Then next step is to setup the authentication settings for it. In this example, I used a previously created `crashplan` user account just for these purposes.
![](/content/images/2016/Aug/03-chap.PNG)
Then on the next page we need to provision the LUN (Logical Unit Number) with a name, where we are creating its space from (the hard drives in the NAS) and then what capacity it will have.
![](/content/images/2016/Aug/04-lun.PNG)
Then we get a summary screen just to confirm everything we decided, and save it all.
![](/content/images/2016/Aug/04-summary.PNG)
<br/>
#### Add the iSCSI Drive in Windows ####
Now that everything is setup on the NAS side, we need to configure everything on the Windows side. So for that, you need to open up Administrator Tools, and then find iSCSI Initiator in the list of tools. Open it and you will see a screen like this:
![](/content/images/2016/Aug/05-iscsi.PNG)
In here you need to click the Discovery tab at the top, then click the `Discovery Portal` button
![](/content/images/2016/Aug/06-discover.PNG)
This will bring up a popup for you to enter in the IP address and iSCSI port for your NAS, which would be 3260 unless you changed it earlier.
![](/content/images/2016/Aug/07-ip.PNG)
Then click OK and once you have done that, you should now have a discovered target in your list that is your NAS, like so:
![](/content/images/2016/Aug/08-targets.PNG)
Now you simply click `Connect` to establish a connection with it, which will show you a screen like this, where you need to click the `Advanced` button.
![](/content/images/2016/Aug/09-connect.PNG)
In here you will check the box to enable CHAP log on, and then fill it in with the login info you specified earlier on the NAS side.
![](/content/images/2016/Aug/10-chaplogon.PNG)

Then click Okay to save that and you should be greeted with this prompt that you need to initialize a new disk:
![](/content/images/2016/Aug/11-newdisk.PNG)
So click OK on that and then inside Disk Management, find your new Disk, then right click on it and create a new Simple Volume for the space it is allotted and give it a name so you know what it is.
![](/content/images/2016/Aug/12-newvolume.png)
I named mine `Crash Plan NAS` to make it super obvious. Once you have done that then when you open up `My Computer` you should see a new drive there, like so:
![](/content/images/2016/Aug/13-mycomputer.PNG)
<br/>
#### Add iSCSI Drive to Crash Plan ####
The last part of the process now that we've gone through all the setup, is to now tell Crash Plan to start backing up to that new iSCSI drive we created.

For this you just need to open up the Crash Plan application, go to your list of Destinations, then click the Folders tab where you will see the existing folder destinations you have.
![](/content/images/2016/Aug/14-crashplandestination.PNG)
Once here, then you simply need to click the Select button to open a prompt for picking a new destination, which in my case is a Crashplan folder on the iSCSI drive as you can see here:
![](/content/images/2016/Aug/15-crashplan-folder.PNG)
Once you've picked that then the dialog will be dismissed and you simply click `Start Backup` and it will start backing up to your NAS.

Tada! All Done!

Now so long as you have Crash Plan running on your PC and your QNAP NAS turned on, it will be backing up now to your external NAS so that you have a second copy of all your precious files.