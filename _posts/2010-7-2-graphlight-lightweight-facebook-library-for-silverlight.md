---
layout: post
title: GraphLight - Lightweight Facebook library for Silverlight.
permalink: /blog/graphlight-lightweight-facebook-library-for-silverlight
category: Blog
date: 2010-07-03 05:12:02
modified: 2010-07-03 05:12:02
author: admin
id: 503e6fb0-6cbc-4749-8589-26d222ace391
---

If you are developing applications for Facebook, then you know that the old Facebook REST API has been replaced
with the new Graph API. To make sure that my Facebook applications keep working, and considering the uncertain
future of the Facebook Developer Toolkit, I decided to write my own Graph SDK - GraphLight.

In designing GraphLight, my major goals were to make it small and simple to use, and to be compatible with the
asynchronous programming model supported by Silverlight. After some experimentation, I decided to use
[Reactive Extensions Library](http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx) to make the most of the
asynchronous nature of Silverlight.

GraphLight is very easy to use, as I will demonstrate with the following samples:

First, let’s initialize GraphLigh by providing it with a valid access_token and then get information about the current user:

{% highlight C# %}
    Profile Me;
    public MainPage_Loaded()
    {
        GraphApi.access_token = access_token;
        GraphApi.Me.Subscribe(OnMe);
    }

    private void OnMe(Profile profile)
    {
        me = profile;
        name.Text = me.name;
        about.Text = me.about;
        pic.Source = new BitmapImage(new Uri(me.Picture));
    }
{% endhighlight %}

Next, let’s get the list of friends:

{% highlight C# %}
    Me.Friends.Subscribe(
        freinds =>
        {
            foreach (Profile friend in friends)
                // do something application specific
        }
    );
{% endhighlight %}

Uploading a photograph is also quite easy:

{% highlight C# %}
    album.Upload(“Me wrestling with polar bear”, photoStream)
         .Subscribe(pid => status.Text = "Upload succesful pid=" + pid);
{% endhighlight %}

You can use reactive extensions to simplify complex asynchronous scenarios. For example, you can use ForkJoin
to wait for all asynchronous downloads to finish, and then call the subscriber with the results:

{% highlight C# %}
    List<Photo> allPhotos = new List<Photo>();
    albums.Where(a => a.name != "Profile")
          .Select(b => b.Photos)
          .ForkJoin().Subscribe(
               v =>
               {
                   foreach (var photos in v)
                   {
                       allPhotos.AddRange(photos);
                   }
               }
          );
{% endhighlight %}

Fair warning: GrahLight is not a fully supported, take-care-of-it-all library. If you like to tinker with your code
and modify your library to meet your needs, then GraphLight could be for you. I would also recommend looking at
similar libraries on [CodePlex](http://www.codeplex.com/), for example [Facebook Graph Toolkit](http://facebookgraphtoolkit.codeplex.com)

 * [Download GraphLight project](http://dl.dropbox.com/u/3528765/GraphLight.zip)
 * [Comic Composer](http://apps.facebook.com/comiccomposer)
 * [Example of OOB Facebook application](http://dl.dropbox.com/u/3528765/samples/Graph.html)
