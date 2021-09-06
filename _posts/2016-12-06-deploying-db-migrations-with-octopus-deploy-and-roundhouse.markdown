---
layout: post
title: Deploying SQL Database Migrations with Octopus Deploy and RoundhousE
date: '2016-12-06 20:26:55'
tags:
- roundhouse
- octopusdeploy
- database-migration
- deployment
- sqlserver
---

For those of you living under a rock in the .NET deployment ecosystem for the last few years, a hot contender has come out and become my (and many others) go-to tool for facilitating deployments, and that tool is [Octopus Deploy](http://octopus.com).

Octopus Deploy handles deploying your .NET websites and services with ease, and as time has gone on, they have made it easy to push out your database changes in that same deployment pipeline with a variety of tools. But one tool absent from their [current documentation](http://docs.octopusdeploy.com/display/OD/SQL+Server+databases) (**Update:** *not anymore! This post has since been added*) and search engine results, is pretty cool little tool called [RoundhousE](https://github.com/chucknorris/roundhouse/), so I plan to rectify that with this post.

## Why RoundhousE?

I prefer to use RoundhousE currently for database migrations over other tools simply for the fact that you facilitate migrations by creating actual sql change scripts that get checked into your repo, so you know the exact same scripts get run on each environment involved in your pipeline. Contrast with with other database migration tools that will instead do a diff at deployment time between the way your database is structured and how it thinks it should be structured and then apply a script to make those changes. The only way you get to know what will actually get changed on a deployment is to then actually do the deployment and hope that there haven't been any small changes you didn't know about that might cause the deployment to fail, or act not like you expected.

## Okay enough on the why, lets get to work

To get started I will assume that you already have a .NET website project that you have being packaged up in some way, that you have Octopus Deploy setup to be deploying that website, and that you've added in RoundhousE to handle your database migrations locally for development purposes. If you don't, then go read up on the RoundhousE and Octopus Deploy documentation, get all that setup and then come back once you've done that. I promise it's worth it!

## What all are we doing?
Just so we don't get ahead of ourselves, here is an overview of what we will be doing in three simple steps:

* **Getting Everything Packaged Up**
* **Make Our Package Deployable**
* **Add Step to Deploy Changes**

Okay so let's get started!

### Getting Everything Packaged Up  

The first thing we need to take care of is figuring out a way to get our RounhousE migration scripts packaged up so we can later get them included with all out other packages that Octopus Deploy takes care of pushing out for us. We can accomplish this by creating an empty project with a cool name like DbDeployment (or whatever you choose) that will sit along side all the other projects in our solution like so:

![](/content/images/2016/Aug/dbdeployment.png)

Now that you have this new project, just edit the DBDeployments csproj file and include an item group like so, with the relative path to your database migration scripts matching where you have yours stored in your solution folder structure

`<ItemGroup>
    <Content Include="..\..\src\DatabaseMigration\**\*.sql">
      <Link>DatabaseMigration\%(RecursiveDir)%(FileName)%(Extension)</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
</ItemGroup>`

This will include the migration scripts you already have in the folder structure you have them in already, so you don't have to remember to go and add each new script manually to your project each time, or have to make a second copy in another location to be brought into your project.

Next we need to add in the RounhousE executable so it gets packaged up to deploy those migration scripts for us. In that already open csproj file, add in another item group like what I have below, pointed to where you have your executable

`  <ItemGroup>
    <Content Include="..\..\tools\roundhouse\rh.exe">
      <Link>rh.exe</Link>
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
</ItemGroup>`

Now whenever you call in your build script to package up your solution with OctoPack, you should now have another package titled DbDeployment.nupkg that will contain your SQL script folder with the scripts inside it and the rh.exe, but that's not quite enough to get things deploying.

### Make Our Package Deployable

So now that we have things getting packaged up, we need to facilitate it being deployed. To do that, all we have to do is add in a Powershell(or C#, F#, or Bash) script that Octopus Deploy picks up on based on a [naming convention](http://docs.octopusdeploy.com/display/OD/Custom+scripts#Customscripts-ScriptsinPackages), and then it will deploy the package, in whatever manner we tell it to do. In this case we will be adding in a Powershell script named Deploy.ps1, which tells Octopus Deploy to run the contents of the script as the deployment for this package. So create a deploy.ps1 file as part of our DBDeployment project, and fill in its contents like so

<script src="https://gist.github.com/hulahomer/2c51cc2c4acc26493354deda3544499a.js"></script>

In this script you can see we say to look for the RounhousE executable at the base of the package, along with where to find the migration scripts, and where to output the log to. Then next you can see we are pulling in some parameters from Octopus Deploy with the $OctopusParameters notation, which if you do not need, you can remove, but these can be handy for when you could have scripts to run for different environments, or if the database server and name are different for each instance. Then lastly we output where we will be running RounhousE, and then actually run it.

Once you have this file in place, your DbDeployment package should look more like this in Visual Studio:

![](/content/images/2016/Aug/all_items.png)

Then when you re-run your build to OctoPack everything in your solution, you will seeing a DbDeployment package created (on the left) with contents similar to whats seen below on the right

![](/content/images/2016/Aug/packaged_up.png)

Voila! Now we just need to get this stuff deployed, so lets get to it!

### Add a Step to Deploy Changes

Thanks to how easy Octopus Deploy is to use, this part is super easy, and all it involves is adding a simple step to your existing deployment process to now include this new package of database changes.

To start, go into your existing project in Octopus, head on over to its `Process` page, then simply click the add a step button. This step will be the step type of `Deploy a Package` and you can name it whatever you want, like `Deploy Db Changes` or `Deploy Database Changes` or maybe even `Ben's cool db step`, your choice. But once you add it, then you will need to fill in what role your database server is a part of in Octopus Deploy and then tell it where to find your DbDeployment package, whether that's on the built in feed or an external feed you are using. It should end up looking something similar to this:

![](/content/images/2016/11/deploydbchanges_step.png)

Then go and save that new step, and now, one you create a new release and choose to deploy it, you will see your new step get executed and have some sort of output similar to this:

![](/content/images/2016/11/dbscripts_run.png)

And congrats! You now have your database changes that you run via RoundhousE locally, also now running the exact same way when you deploy code to your servers!
