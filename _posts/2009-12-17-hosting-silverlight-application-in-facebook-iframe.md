---
layout: post
title: Hosting Silverlight application in Facebook iframe
permalink: /blog/hosting-silverlight-application-in-facebook-iframe
category: blog
date: 2009-12-17 17:24:30
modified: 2010-04-21 07:38:03
author: admin
id: 22acb42f-250a-4a24-8684-9eb3a41640a7
---

If you’re building a Silverlight application for Facebook, you have two
choices: (1) build a stand alone application and use Facebook Connect
for authentication, or (2) create a canvas application that will appear
on a Facebook site and use the parameters passed to iframe to establish
a session. The former is well covered in the Facebook Toolkit
documentation, but I couldn’t find any examples of how to embed
a Silverlight application within a Facebook iframe. It took a few days
worth of research and experiments to get it right and I hope this post
will be helpful to anyone who is working on Silverlight canvas
application for Facebook.

### Embedding Silverlight into Facebook canvas

I know three ways of embedding Silverlight application into a Facebook
canvas.

(1) Select the IFrame option for Render Method in Canvas section of the
application settings and embed the Silverlight application inside the
iframed page using standard methods (object tag, silverlight.js, etc.).
(2) Use `<fb:iframe>` tag in fbml based canvas. (3) Use fbml tag
`<fb:silverlight>`. The `<fb:silverlight>` tag would be the preferred
method, but according to the Facebook wiki, the tag is not implemented
at this time. That leaves only options 1 and 2 available – embed
Silverlight in iframe. Whether you go with IFrame canvas or use
`<fb:iframe>` tag in fbml, the following steps are the same:

### Extracting parameters passed by Facebook to iframe page

First lets take a look at iframe created by Facebook

    <iframe frameborder="0" scrolling="no" name="fb_iframe_4b2894**"
      class="smart_sizing_iframe" smartsize="true"
      src="http://mysite.com/facebookapp/?fb_sig_in_iframe=1&
          fb_sig_iframe_key=c***f&
          fb_sig_locale=en_US&
          fb_sig_in_new_facebook=1&
          fb_sig_time=1***7&
          fb_sig_added=1&
          fb_sig_profile_update_time=1256693703&
          fb_sig_expires=1261108800&
          fb_sig_user=1***1&
          fb_sig_session_key=2***1&
          fb_sig_ss=b***&
          fb_sig_cookie_sig=f***b&
          fb_sig_ext_perms=status_update,photo_upload,publish_stream&
          fb_sig_api_key=e***6&
          fb_sig_app_id=2***0&
          fb_sig=0***2"
      style="width: 758px; height: 679px;"/>

As you can see, Facebook adds a bunch of parameters prefixed with
fb_sig to the url. We don’t need to worry about all of them. The most
important parameters needed to establish a connection to Facebook are
fb_sig_ss, fb_sig_session_key and fb_sig_api_key.

The way you extract url parameters depends on which technology you use.
I am using asp.net mvc for my development, but it should be trivial
to change the code to work with plain asp.net or php.

    public class FacebookController : Controller
    {
        public ActionResult Index(
                string fb_sig_added, string fb_sig_user, string fb_sig_ss,
                string fb_sig_session_key, string fb_sig_api_key)
        {
            ViewData["ss"] = fb_sig_ss;
            ViewData["key"] = fb_sig_session_key;
            ViewData["apikey"] = fb_sig_api_key;
            ViewData["user"] = fb_sig_user;
            return View();
        }
    }

Values passed by Facebook to our iframe was saved in ViewData, so we
can pass them to the Silverlight application later. One value we do not
pass to the application is fb_sig_added. The fb_sig_added parameter is
important since it indicates if a user has authorized the application
or not. If fb_sig_added is “0”, that means a user has not authorized
the application and the rest of the fb_sig_ parameters will be empty.
Only a limited set of Facebook APIs can be accessed by an unauthorized
application, and I am not going to focus on that scenario.

### Passing parameters to Silverlight application

Below is a sample of a striped down object tag with InitParams used to
pass Facebook parameters to the Silverlight application.

    <object data="data:application/x-silverlight-2,"
       type="application/x-silverlight-2" width="100%" height="100%">
        <param name="source" value="/MyApp.xap"/>
        <param name="InitParams" value="key=<%= ViewData["key"] %>,
          apikey=<%= ViewData["apikey"] %>,ss=<%= ViewData["ss"] %>,
          user=<%= ViewData["user"] %>" />
    </object>

### Retrieving values from InitParams

To retrieve values passed in InitParams, we need to add a bit of code
to the Application_Startup handler in the App class.

    private void Application_Startup(object sender, StartupEventArgs e)
    {
        if (e.InitParams.ContainsKey("ss"))
        {
            FacebookFx.SessionSecret = e.InitParams["ss"];
        }
        if (e.InitParams.ContainsKey("key"))
        {
            FacebookFx.SessionKey = e.InitParams["key"];
        }
        if (e.InitParams.ContainsKey("user"))
        {
            FacebookFx.UserId = Convert.ToInt64(e.InitParams["user"]);
        }
        if (e.InitParams.ContainsKey("apikey"))
        {
            FacebookFx.ApiKey = e.InitParams["apikey"];
        }
        this.RootVisual = new Page();
    }

### Creating Facebook session and retrieving list of friends

The last step is to use parameters we received from Facebook to
establish a session and make a call to retrieve the list of friends
of our current user.

    static public void Init()
    {
        Facebook.Session.BrowserSession _session = 
            new Facebook.Session.CachedSession(ApiKey, SessionKey, SessionSecret); 
        _session.UserId = UserId;
        Facebook.Rest.Api _api = new Facebook.Rest.Api(_session); 
        _api.Friends.GetAsync(GetFriends, this);
    }

    static public void GetFriends(IList<long> friends, object state, FacebookException e)
    {
        if (e == null)
        { 
            List<long> fs = new List<long>(friends);
            …
        }
    }

### Getting more information

Silverlight is the new kid on the block as far as Facebook is concerned.
Fortunately, many of the same issues concerning Silverlight apply to
Adobe Flash as well. For example, I was able to find several articles
describing process of embedding Flash application into a Facebook
iframe, and other than minor differences in how parameters are passed
from web page to the application, the information was still relevant to
Silverlight.

* <a href="http://wiki.developers.facebook.com/index.php/Authorizing_Applications">Facebook: Authorizing Applications</a>
* <a href="http://facebooktoolkit.codeplex.com/">Facebook Developer Toolkit</a>
