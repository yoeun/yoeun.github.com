---
layout: post
title: "Remote access to a self hosted Nancy web service"
description: ""
category: 
tags: []
---
{% include JB/setup %}

If you've only ever run [Nancy](http://nancyfx.org) through Visual Studio or IIS, you will likely have never run into this problem. But if you try to run Nancy under a non-Administrator account--for example, self hosting Nancy in a Windows service running under Network Service--you'll get a nasty little surprise: 

    System.Net.HttpListenerException (0x80004005): Access is denied
       at System.Net.HttpListener.AddAllPrefixes()
       at System.Net.HttpListener.Start()

#### Problem: 

Windows does not allow non-Administrator accounts to run HTTP listeners on arbitrary ports. In this case, port 4567.

    nancyHost = new Nancy.Hosting.Self.NancyHost(new Uri("http://localhost:4567"));
    nancyHost.Start();

#### Solution:

You will need to open up the port used by Nancy. 

    netsh http add urlacl url=http://+:4567/ user="NT AUTHORITY\Network Service"

Try your program again and it should now work. 

Reference: [Stack Overflow](http://stackoverflow.com/questions/8548678/remote-access-to-a-nancy-self-host)