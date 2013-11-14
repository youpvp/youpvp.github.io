---
layout: post
title: Silverlight, meet Shell.Application
permalink: /blog/silverlight-meet-shellapplication
category: blog
date: 2010-01-01 16:09:46
modified: 2010-06-03 02:17:03
author: admin
---

Silverlight 4 is coming, and with it a controversial ability to call local
automation objects. Below I will review some of the standard automation
objects available in Windows that can be used from Silverlight 4.

### Shell.Application

Shell Automation enables access to features of Windows shell. This includes
displaying standard dialogs, arranging windows, starting applications, starting
and stopping services, and more.

Probably the most useful function of Shell.Application is 
<a href="http://msdn.microsoft.com/en-us/library/bb774148(VS.85).aspx">ShellExecute</a>.
Using ShellExecute, it’s possible to run executable or open or print a document.

    dynamic shell = AutomationFactory.CreateObject("Shell.Application");
    shell.ShellExecute("notepad.exe", "", "", "open", 1);

<!-- more -->

### WScript.Shell

<a href="http://msdn.microsoft.com/en-us/library/ahcz2kh6(VS.85).aspx">Windows Script Host Shell Object</a>
provides access to many useful system level functions, most noteworthy being
registry access (RegRead, RegWrite, RegDelete), launching executables (Run, Exec),
simulated user input (SendKeys).

    dynamic wscript = AutomationFactory.CreateObject("WScript.Shell");
    string user = (string)wscript.RegRead(
         "HKCU\Software\Microsoft\Office\Common\UserInfo\UserName");

### WScript.Network

<a href="http://msdn.microsoft.com/en-us/library/907chf30(VS.85).aspx">Windows Script Host Network Object</a>
provides access to information about network resources, such as network drives
and printers as well as user name, domain name, and computer name.

### Scripting.FileSystemObject

<a href="http://msdn.microsoft.com/en-us/library/d6dw7aeh(VS.85).aspx">File System Object</a>
gives scripting languages an ability to access file system. While Silverlight
has a managed version for many functions provided by File System Object, its other
functions can still be useful.

### Microsoft office applications

All Microsoft Office applications support automation and is primarily the reason
for com automation interop’s introduction in Silverlight 4. Version independent
program ids follow a simple pattern of (Office Application Name).Application
(i.e., Word.Application, Excel.Application, etc). I found another office automation
object called OfficeCompatible.Application, but I couldn’t find any useful
information regarding its purpose.

### More…

Some other standard automation objects worth investigating:
<a href="http://msdn.microsoft.com/en-us/library/ms630827(VS.85).aspx">Windows
Image Acquisition</a> (WIA.DeviceManager, WIA.ImageFile, WIA.ImageProcess),
<a href="http://msdn.microsoft.com/en-us/library/aa384006(VS.85).aspx">Task
Scheduler</a> (Schedule.Service), <a href="http://msdn.microsoft.com/en-us/library/ms690288(VS.85).aspx">FaxServer</a>
(FaxComEx.FaxServer), IMAPI2, <a href="http://msdn.microsoft.com/en-us/library/aa752084(VS.85).aspx">Internet Explorer</a>
(InternetExplorer.Application), Shell.Explorer.

Third party applications can also have automation interfaces, like Skype,
for example.

If you would like to find more automation objects, try OleView application
that comes with Visual Studio. One thing to remember is that not every com
object with an IDispatch-based interface can be accessed from Silverlight.
Many com objects are ActiveX controls, and require special support from the
hosting environment to work. Because Silverlight is not a control host, it
will not be able to use ActiveX controls.
