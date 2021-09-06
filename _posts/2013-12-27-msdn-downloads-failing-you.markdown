---
layout: post
title: MSDN downloads failing you?
date: '2013-12-27 05:27:54'
---

Well after talking with a fellow Headspringer about this issue today, I decided that it wasn't a fluke that it happened to me, so I should blog about it.

So I just built a new desktop computer (blog post coming soon on that) and I naturally went to MSDN to download the ISO for Windows 8.1 to start with. This simple task turned into hours of frustration due to some unknown issues with the site, to the point where I was never successfully able to download the ISO and was sitting here with a computer fully put together, but I could do nothing with it.

Eventually I remembered that MSDN used to have some sort of [download manager](http://transfers.one.microsoft.com/ftm/) that could be used, but I couldn't find out why it wasn't trying to use it on this 3.1GB download. So I went to the Google (like ya do), and sure enough found out there was a few things you had to do to actually get the ActiveX control to launch.

To get it working you have to:
* If you are on IE10/11, get IE to be all old and IE9 like by setting it to IE9 in the document mode of the Developer Tools
* Lower your security settings temporarily (Internet Options -> Security tab) and/or add msdn to your trusted sites

Once doing this, the download manager launched when I clicked the download link, and an hour or so later (with a few restarts of the download by the manager), I had a Windows 8.1 ISO that I could now use!