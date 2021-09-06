---
layout: post
title: Cleaning up after MSBuild and Octopus Deploy
date: '2016-11-12 00:31:11'
tags:
- octopusdeploy
- msbuild
- net
---

If you are currently deploying your .NET solution with Octopus and you utilize a CI server like TeamCity, then you might not realize that you could be eating up disk space thanks to the way Octopus Deploy creates packages to deploy and how MSBuild cleans up (or really doesn't) files.

###Do what now?
When you call OctoPack to package up your project, it creates the package in your projects output folder, which is typically going to be `\bin` or maybe specifically `\bin\Release`, so that it can then copy it to some file share somewhere, or maybe to a nuget feed that you have specified. BUT in either of those cases, it will still leave that .nupkg file behind after it completes that operation. This doesn't sound too harmless that it leaves it around since you clean things out before you package up each new build, but that is dependent on your clean process actually cleaning out everything, which in the case of MSBuild (calling `MSBuild /t:clean` ) isn't true.

###Oh brother...
Yup. If you didn't know that, now you do. When you call `msbuild /t:clean` to get it to clean your project, it will delete all files in the output folder that it knows it created and nothing else. So anything else it doesn't know about that gets dumped in there, like say, Octopus nupkg files, it doesn't touch them. So then you end up with an output folder that looks a little something like this (except multiple the number you see here by 10, for just this one project):
![](/content/images/2016/11/screenshot_edit.png)

###Now what?
<iframe width="560" height="315" src="https://www.youtube.com/embed/DtRNg5uSKQ0" frameborder="0" allowfullscreen></iframe>

To fix it you can edit your projects .csproj file and add this target for cleaning up the entire output directory when the MSBuild Clean task is run
`
  <Target Name = "Clean">
	<RemoveDir Directories='$(OutputPath)' />
  </Target>
`

Voila! Now you might get a few megabytes of disk space back, or in the case of the long running project I'm on when I discovered this, a few gigabytes.