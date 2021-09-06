---
layout: post
title: When SQL Full Text Search Boggles The Mind
date: '2017-07-25 14:34:55'
tags:
- sqlserver
---

If you go the route of utilizing SQL Server Full Text Search and Indexing capabilities for your application but you don't fully understand all the ins and outs of it, there might be a few issues you run into that seem impossible, until someone points out the super easy solution to you of course.

<br/>
#### The problem ####

Recently on a project, we had a search page that let you search for people in the application by their name, email, or membership number, and a bug was reported with it saying that relevant results weren't coming up when searching for a specific persons whole name. If you searched for just their last name (Smith) we would get a lot of results and they would be somewhere in the list of hundreds to filter through, but when they searched for their whole name to be more precise (Will Smith) they would be completely missing from the list, even though we could verify they were in the database and were the only one with that full name.

Now if you are smarter than me with SQL Full Text Search you might have already picked up on what the issue is, but I didn't for quite a while, until someone else pointed out to me it was the persons name causing the issue.

Huh?

What I mean by that is the persons first name of `Will` is specifically causing the issue.

Again... Huh?

When you create a Full Text Index to use for Full Text Searching (in a view in our instance), one of the last things in the query that you might not always notice, is you specify a StopList like so:

<script src="https://gist.github.com/hulahomer/06255e7b5a4ee5228035af68db7c6c58.js?file=DefaultFullTextIndex.sql"></script>

For those that don't know (like I didn't), a stoplist is a collection of stopwords (like 'a', 'the', or 'is') that are excluded from searching to try and return more relevant results when utilizing a full text index.

The reason this is important is because if you pull up the list of stopwords in the default stoplist, you notice that `Will` is in that list.
<script src="https://gist.github.com/hulahomer/06255e7b5a4ee5228035af68db7c6c58.js?file=LookForStopWord.sql"></script>
![](/content/images/2017/07/Screen-Shot-2017-07-24-at-12.24.49-PM.png)

Oops.

#### The Solution! ####

To solve this quickly and easily we just need to use a new stop list for our index that doesn't have those "bad words" in it anymore. To do this, you can write some SQL like this to create a new stoplist from the default one:

<script src="https://gist.github.com/hulahomer/06255e7b5a4ee5228035af68db7c6c58.js?file=CreateNewStopList.sql"></script>

Then drop the "bad words" from your new stop list:

<script src="https://gist.github.com/hulahomer/06255e7b5a4ee5228035af68db7c6c58.js?file=DropWordsFromStopList.sql"></script>

Then modify (or drop/recreate) your full text index to use the new stop list instead:
<script src="https://gist.github.com/hulahomer/06255e7b5a4ee5228035af68db7c6c58.js?file=RecreateWithNewStopList.sql"></script>

And that's it! Simple as that right?

For a more thorough and complete solution, you instead might want to actually look at entries in the default stop list beyond `Will` and also go ahead and remove those as well from your custom stop list.

<br/>
Now if I could only have told past me about this before I spent so much time pulling out my hair over this bug.