---
layout: post
title: Protecting your Silverlight application from a hijacking
permalink: /blog/protecting-silverlight-application-form-hijacking
category: blog
date: 2010-07-14 18:15:16
modified: 2010-07-15 22:12:40
author: admin
id: 2a9248dd-57b1-44a4-a6bc-1c5d61b55a20
---

First, let me clarify what I mean by “application hijacking.” The traditional definition of hijacking is the following:
*“To seize control of a vehicle by use of force, especially in order to reach an alternate destination.”* By modifying the
definition to make it appropriate for web applications we get something like this: Copying or embedding an application in a way
that is not intended or permitted by the owner, with the goal of attracting users to the hijacker’s website. This issue is
not new, nor is it exclusive to applications. Sites have being battling “deep linking,” copying of images, and other copyrighted
materials for a long time now. Fortunately, a Silverlight application can be hardened to resist hijacking in a way that image
or other passive content cannot.

Below, I review different forms of application hijacking and explain how to detect them.

### Xap hijacking

This most brazen form of application hijacking is done by downloading an application file (.xap) and placing it on the
hijacker’s server. Guarding from this type of hijacking is important because it cannot be detected by analyzing web server logs.

Luckily, detecting xap hijacking is pretty easy. Simply compare the location reported by **App.Current.Host.Source** to your own.
If the location is not the one you expect then, your application has been hijacked. Comparing domains should be sufficient for most cases.

    if (App.Current.Host.Source.Host != "mydomain.com")
        //sound the alarm!
	
### Application “deep linking”

This happens when an application is embedded into the page of an unauthorized website (using <object> tag or silverlight.js) while
pointing to the original xap file. This is probably the most common form of unauthorized embedding. It requires very little effort
on the part of the hijacker, and leaves the owner with the bill for consumed bandwidth.

To detect if an application is deep-linked, check **HtmlPage.Document.DocumentUri** to see if the hosting page is located on the
authorized domain. Watch out for <a href="http://msdn.microsoft.com/en-us/library/2asft85a(v=VS.95).aspx">InvalidOperationException</a>!
If you get it while trying to access HtmlPage’s properties, that means that the host webpage and your application are being served
from different domains. See <a title="Security Settings in HTML Bridge" href="http://msdn.microsoft.com/en-us/library/cc645023(v=VS.95).aspx">
Security Settings in HTML Bridge</a> for explanation. If you want to perform a deep-linking check on your own application hosted from
a different domain (for example CDN), then you need to add the *enableHtmlAccess* parameter to your <object> tag.

While working on sample application, I noticed that xap files hosted on dropbox.com appear to be immune to deep-linking.
I am not 100% sure why, but my current theory is that it is due to Content-Type set to “application/octet-stream.” You can also
use more traditional techniques to fight deep linking, such as checking http referrer.

### Iframe embedding

This situation is not specific to Silverlight apps, but to web pages in general. The solution to fight iframe embedding is often
called “frame busting.” To bust out of iframe, all you need to do is to put the following javascript code in you page:

    <script type="text/javascript">
    if(top.location != location)
    {
       top.location.href = document.location.href;
    }
    </script>

### Deterrents

If you detect that your application is hijacked, you have a few choices:

* Disable application without any explanation to the user
* Show user a message with hyperlink button offering to go to the application’s original site
* Keep working as if nothing happened, but send information about the hijacker’s website to your own web service
  (think of it as a Lojack for your app).

Whatever measure you choose, be considerate to your users. They are not the ones responsible for your application hijacking.

### Conclusion

All this work wouldn’t be worth much if a hijacker can disassemble your application, and easily locate and neutralize
your countermeasures. That is why I would recommend to always obfuscate your application if you are going to place
anti-hijacking code in it.

It is worth remembering that embedding is not always bad; in fact some websites rely on embedding in their business
model and actively encourage it. If your application is a popular target for embedding, you may want to consider
making use of it rather then trying to fight it.
