---
layout: post
title: Facebook announces its first official .Net SDK
permalink: /blog/facebook-announces-official-net-sdk
category: Blog
date: 2010-07-17 12:35:20
modified: 2010-07-17 12:37:19
author: admin
id: b42dd1d2-6de1-4a7c-ac90-ac1b2c81c742
---

For quite some time now, .NET developers have been left out in the cold without official support from Facebook.
No more, my friends! Today, [Andrey Goder](http://www.facebook.com/andreygoder) announced the first release of Facebook’s
[official C# SDK](http://developers.facebook.com/blog/post/395) in his blog post. For many of us, this raises the following questions:
what does it mean for all the unofficial .Net SDKs floating around?
Do I keep using my own Facebook library or should I switch to the official SDK?

The decisive factor in answering this question for all Silverlight developers is, of course, Silverlight support.
Based on the first look, the official SDK is not Silverlight ready, although it should be easy enough to add support for an asynchronous
communication model.

Also, Facebook SDK is quite similar to [GraphLight](/blog/GraphLight), in that neither provides any help with Facebook authentication.
All in all, the alpha state of the first official .Net SDK is pretty obvious. For now, I will stick with my own [GraphLight](/blog/GraphLight),
but I will keep an eye on the Facebook SDK to see how its Silverlight support progresses.
