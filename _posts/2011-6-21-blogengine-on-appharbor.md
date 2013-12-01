---
layout: post
title: Hosting BlogEngine  blog on  AppHarbor
category: blog
date: 2011-06-21 21:28:45
author: admin
---

I’ve being thinking about giving cloud hosting a try since Azure went into beta. But, the price of a single instance on Azure with
a SQL database makes it completely impractical for personal blog. Good news: you can now jump into cloud without breaking the bank.
Mountain View startup <a href="http://appharbor.com">AppHarbor</a> will give you one instance with 20MB SQL database for free!
AppHarbor cloud hosting comes with a twist -- instead of a traditional deployment mechanism than you locally compile your project,
build the deployment package, and push it to the cloud, AppHarbor gives you a <a href="http://git-scm.com/">Git</a> source repository
from which it builds the solution on every commit, and deploys results of the successful builds to the site.

When I started the process of deploying BlogEngine on AppHarbor, I had some doubts about whether BlogEngine could work in the cloud
environment. I can now say that it can, with some minor caveats. Due to the transient nature of application deployment in the cloud,
all data should be stored in the database, external CDN, or must be checked into the source repository. If you want to embed image
into your blog post, you must upload it to the external CDN first, and then link that location from your article. You can use
<a href="http://www.dropbox.com/">DropBox</a> as CDN as long as your site traffic is light, or you can get cloud storage for cheap
from Amazon or Azure.

Installing BlogEngine on AppHarbor was surprisingly easy; in all, it took about two hours including downloading BlogEngine’s sources,
installing the Git client, reading instructions, and going through a few failed builds. Here is a brief overview of the steps I made
to get a copy of my blog deployed to AppHarbor:

* Download BlogEngine’s source code from <a href="http://blogengine.codeplex.com/releases">Codeplex</a>
* Move lib directory to BlogEngine directory and update references in the BlogEngine.Core project. (*It is quite possible that if you
  don’t follow this step, everything would work just fine, but I did it, so I mention it here.*)
* Copy SQLServer.NET_4.0\_Web.Config file from the BlogEngine.NET\setup\SQLServer directory (or if you prefer to go with MySQL database,
  MySQL.NET_4.0\_Web.Config from BlogEngine.NET\setup\MySQL) to BlogEngine.NET directory and rename it to Web.config (delete existing
  web.config first)
* Update target framework in the BlogEngine.NET project properties from .NET Framework 3.5 to .Net Framework 4. (Without this step,
  the build failed for me.)
* Create new account on AppHarbor.com (if you don’t have one already)
* Create new <a href="https://appharbor.com/application">application</a>
* Add database to your application. Select SQLServer or MySQL depending on the web.config file you chose in the step above. Set
  connection string name to ‘BlogEngine’
* Connect to the database with SQL Server Management Studio (or MySQL equivalent) using information provided on the database page.
  Load MSSQLSetup2.0.0.0.sql (or MySQLSetup2.0.0.sql) and execute it
* Go back to your application page and follow the instructions on how to check in BlogEngine sources using Git client (get git client
  first if you don’t have one)
* At this point build should successfully complete and you should be able to follow the link to your application and see BlogEngine’s
  front page
* Follow instructions on the BlogEngine site to configure you fresh install
* Import existing posts using BlogML and you’re done

One bit of strange behavior that I noticed was that absolute links would fail to load, due to the requests being made on a port other
than 80. Apparently this is a <a href="http://support.appharbor.com/kb/getting-started/workaround-for-generating-absolute-urls-without-port-numbe">
known issue</a> that I’ve fixed by modifying AbsoluteWebRoot property in Utils.cs.