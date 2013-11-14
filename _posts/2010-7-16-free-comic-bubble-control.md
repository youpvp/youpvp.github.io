---
layout: post
title: Free Comic Bubble Control
permalink: /blog/free-comic-bubble-control
category: Blog
date: 2010-07-16 17:17:49
modified: 2010-07-16 17:17:49
author: admin
id: cdc35cde-2023-4d2e-afab-0064fba9ab88
---

<img alt="free" src="/i/free-comic-bubble-control/free.jpg" width="151" height="244" />

I’ve been working on my Silverlight Comic Creator application for a pretty long time now. One of the more
difficult tasks was creating speech bubble control that is customizable, easy to use and one that looks like
it belongs in the comic. Well, today I am releasing my free Comic Bubble control to the public, so your
application can have cool speech bubbles too without having to do all the heavy lifting.

You can add comic bubble to your Silverlight application in a few easy steps:

1. Download Free Comic Bubble Library

2. Add FreeBubbles.dll to your project references

<img alt="" src="/i/free-comic-bubble-control/ref.jpg" />

3. Add custom namespace to the top parent control

    <UserControl x:Class="BubblesApp.MainPage"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:bubbles="clr-namespace:FreeBubbles;assembly=FreeBubbles"

4. Add Bubble control to your XAML

   <bubbles:Bubble Text="Hello"
          FontFamily="Arial"
          FontSize="22"
          OutlineThickness="2"
          AllowEdit="True"
          TailPoint="30 70" />

5. Customize Bubble’s appearance by changing its properties

### License

Comic Bubble Control is free for use in any **non-commercial** project. Send me a link to your project;
I would love to see the creative ways that this control can be used.

 * [Comic Bubbles demo](http://dl.dropbox.com/u/3528765/samples/BubblesApp.html)
 * [Download Comic Bubble library](http://dl.dropbox.com/u/3528765/FreeBubbles.zip)