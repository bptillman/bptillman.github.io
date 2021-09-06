---
layout: post
title: Virtual Box telling you that you don't have VT-x ?
date: '2014-02-12 04:45:15'
---



But you know your processor does and you need to get a VM up and running.

Well then check to make sure you don't have Hyper-V installed like I did.

By having Hyper-V installed, it blocks detection of your processors VT-x instructions since its running Windows on top of Hyper-V.

So either remove Hyper-V if you don't plan to use it and then everything will be fine in Virtual Box, or use Hyper-V for your VM needs and profit!