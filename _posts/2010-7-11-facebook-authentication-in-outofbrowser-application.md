---
layout: post
title: Implementing Facebook authentication in a Silverlight out-of-browser application
permalink: /blog/implementing-facebook-authentication-in-a-silverlight-application
category: blog
date: 2010-07-11 10:29:03
author: admin
id: 40b7d78f-0ac1-470f-8c87-27b0b178a38d
---

In my previous post about the <a href="/blog/GraphLight-Lightweight-Facebook-library-for-Silverlight.aspx">GraphLight</a> library,
I mentioned that the first step in using the library is to supply it with an access token.
What I conveniently skimmed over is the explanation of what an access token is and how
does one get a hold of it. I did this because answering those questions isn’t simple,
and would benefit from dedicated elaboration.

According to Facebook: “An access token allows an application to perform authorized
requests on behalf of the user by including the access token in <a href="http://developers.facebook.com/api">Graph API</a> requests.”
You can read about the process of obtaining an access token on the <a href="http://developers.facebook.com/docs/authentication/">Facebook Developers Site</a>.
Although Facebook documentation provides a good starting point, it does not account
for peculiarities of Silverlight. In this post, I will go over issues I encountered
while implementing Facebook authentication in a Silverlight OOB application, and
offer my solutions.

Facebook documentation outlines two main flows of authentication available to
applications: The first method utilizes sequence of redirects to Facebook site and
back; the second method uses single sign-on with the JavaScript SDK. Of the two,
only the former method can be used in a Silverlight OOB application due to
restrictions on access to the hosting page’s DOM. Upon closer examination,
the <a href="http://developers.facebook.com/docs/authentication/desktop">Desktop Application Authentication</a>
process emerges as the most obvious choice for an OOB application.

Implementing a desktop authentication flow in accordance with Facebook documentation
is quite straightforward:

1. Create WebBrowser control in code or in XAML
2. Navigate to <em><a href="https://graph.facebook.com/oauth/authorize">https://graph.facebook.com/oauth/authorize</a>…</em>
3. Intercept the redirect to login_succes.html using LoadCompleted event and read the
   access token out of the URL

### Problem #1

WebBrowser control needs **Full-Trust** to show pages from sites other then application’s
origin. Because the Facebook authentication process requires redirect to _facebook.com_,
the OOB application must be in full-trust mode! After many different attempts to find an
acceptable workaround for standard OOB applications, I came to the conclusion that
although it can be done, the resulting user experience is so poor that it is simply not
worth it.

**Solution:** Configure application to require Full-Trust.

### Problem #2

Using the <a href="http://msdn.microsoft.com/en-us/library/system.windows.controls.webbrowser.loadcompleted(VS.95).aspx">LoadCompleted</a>
event to intercept the redirect to login_succes.html doesn’t work because the Uri property
of NavigationEventArgs is always null. If one is unable to read the access token out of
the URL, using the <a href="http://developers.facebook.com/docs/authentication/desktop">Desktop Application Authentication</a>
process is no longer an option.

**Solution:** An alternative approach comes in the form of an authentication flow
recommended for Web Applications and <a href="http://developers.facebook.com/docs/guides/mobile#auth">Mobile Applications</a>.
Web application authentication uses a two-step process, with the first step similar
to that of desktop authentication, but instead of an access token, it returns code
that can be exchanged for an access token in a separate step.

In order to detect redirects and pass code to the Silverlight application, following
steps need to be taken:
1. Create success.html page

{% highlight html %}
<html>
  <head>
   <title>Facebook Login Callback</title>
   <script type="text/javascript">
      <strong><font color="#800000">window.external.notify(window.location.href);</font></strong>
   </script>
  </head>
  <body />
</html>
{% endhighlight %}

2. Success page should be hosted on the same domain as the application and its url
should start with _Connect URL_ (see <a href="http://wiki.developers.facebook.com/index.php/Creating_a_Platform_Application">Facebook connect settings</a>).
Both conditions are <i>required</i> for authentication process to work.

3. Point *redirect_uri* parameter to the location of the success page

4. Add ScriptNotify handler to WebBrowser control:

{% highlight c# %}
browser.ScriptNotify += (a, b) =>
{
    int n = b.Value.IndexOf("?code=");
    if (n > 0)
    {
        //extract code from b.Value
        //exchange code for access token as described in Facebook documentation
        WebClient wc = new WebClient();
        wc.DownloadStringAsync(new Uri(
             string.Format(token_xchange, app_id, redirect_url, secret, code)));
        wc.DownloadStringCompleted += (c, d) =>
        {
            //extract access token from d.Result
            LoginSuccess(access_token);
        };
    }
    else
    {
        if (b.Value.IndexOf("user_denied") > 0)
        {
            LoginFailed();
        }
    }
};
{% endhighlight %}

At last a bit of good news: The method described above actually works, but…

### Problem #3

Doing a token exchange from a Silverlight application requires a secret key
to be stored inside of the package redistributed to the end users, which is
inherently not secure.

**Solution:** Replace success.html with success.aspx or success.php or any
other technology that allows one to execute code server-side. Perform token
exchange on the server and simply pass the access token to ScriptNotify
handler.

<aside>
In an attempt to find a simpler solution, I found a post on the Facebook
developers forums showing a way to get an access token in one step, without
going through a token exchange. Unfortunately, that method relies on
undocumented features and looks kind of “hacky,” so I am not going to
present it here.
</aside>

### Postmortem

Implementing Facebook authentication in an OOB silverlight application
turned out to be a challenge. Most of the issues I encountered were related
to limitations of the WebBrowser control. Some security restrictions were
expected and quite reasonable, while others appear to be to overreaching.
It would be nice to see Microsoft considering use cases for Silverlight
applications needing to login to OAuth based services (Facebook, Twitter,
Flicker…).

### Bonus feature: Logout or “What goes up must come down“

After all the trouble with authentication, it would be nice if logging out
was simple… and it mostly is. Presently the only way to properly log a user
out of Facebook is by redirecting the user to logout.php with app id and
session as parameters. One little problem is how do we get a hold of the
session? Well, as it happens, session is part of the access token and can
be extracted from it with little effort.

{% highlight c# %}
private string logout_format = 
    "http://www.facebook.com/logout.php?app_key={0}&session_key={1}&next={2}";

public void Logout(string access_token)
{
    access_token = HttpUtility.UrlDecode(access_token);
    int s = access_token.IndexOf("|");
    int e = access_token.LastIndexOf("|");
    string session = access_token.Substring(s + 1, e - s - 1);
    browser.Navigate(new Uri(string.Format(logout_format, app_id,
        session, HttpUtility.UrlEncode(OAuthUrl))));
}
{% endhighlight %}

* <a href="http://developers.facebook.com/docs/authentication/">Facebook OAuth 2.0 overview</a>
* <a href="http://dl.dropbox.com/u/3528765/samples/graph.html">Sample Facebook OOB application</a>
