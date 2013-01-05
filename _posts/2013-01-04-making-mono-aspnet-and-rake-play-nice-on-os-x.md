---
layout: post
title: "Making Mono, ASP.NET and rake play nice on OS X"
description: ""
category: 
tags: [mono, rake, asp.net, c#, nancy, osx]
---
{% include JB/setup %}

Fed up with Parallels and Windows heating up and over-working my MBP just to develop [Nancy](http://nancyfx.org/) websites, I decided to explore my options with C# development on OS X. I came across this article to help me get started: [Building a simple Nancy app from scratch using Mono and MonoDevelop in OSX](http://littlegists.blogspot.com/2012/12/building-simple-nancy-app-from-scratch.html).

*If you're not familiar with Nancy, it's a C# micro-framework inspired by Ruby's [Sinatra](http://www.sinatrarb.com/) for quickly and easily building web services and websites. It's awesome! Stop using ASP.NET MVC and Web API right now. Game over.*

Unfortunately, I ran into some issues building Nancy from source. 

#### Problem:

When running `rake mono`, I ran across the following error:

    /Users/ypen/src/nancyfx/Nancy/src/Nancy.Demo.Authentication/Nancy.Demo.Authentication.csproj could not import "$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets"

This happened for all the Demo projects. I had just wiped my machine and installed a fresh copy of Mountain Lion only a few weeks earlier, including the latest stable 2.10.9 version of Mono. More perplexingly, when I open `Nancy.sln` in MonoDevelop, it builds. 

So what's up?

#### Solution:

After some Googling, I found the cause to the problem and a potential solution on the [Ximarin bugzilla site](https://bugzilla.xamarin.com/show_bug.cgi?id=6327#c2).

The Demo .csproj files were looking for a `v10.0/WebApplications/Microsoft.WebApplication.targets` file, but Mono didn't include one for v10.0. It only came with v9.0. MonoDevelop does some magic when it opens .sln files, which is why you only see this issue when building with xbuild directly.

Per the recommendation on bugzilla, I copied the v9.0 directory into v10.0:

    cd /Library/Frameworks/Mono.framework/Versions/2.10.9/lib/mono/xbuild/Microsoft/VisualStudio
    sudo cp -r v9.0 v10.0

I went back to my working copy of Nancy and re-ran `rake mono` and it worked.

Now I can happily develop Nancy websites on OS X. 