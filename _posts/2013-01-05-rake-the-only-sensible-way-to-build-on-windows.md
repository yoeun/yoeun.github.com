---
layout: post
title: "Rake: The only sensible way to build on Windows"
description: ""
category: 
tags: []
---
{% include JB/setup %}

If you're still using NAnt or MSBuild to build your .NET projects on Windows, stop! *You're doing it wrong!* &trade;

Instead, use [Rake](http://rake.rubyforge.org/).

#### What is it?

Rake is the Ruby version of Make and is used to build Ruby projects, such as Rails. It reads a `rakefile` which contains tasks to help you build, test, manage and deploy your code. 

Rake is superior to NAnt or MSBuild because you don't need to mess around with XML. You get a simple-to-use DSL with full access to the entire Ruby language at your disposal. No more hacks to make NAnt do what you want. If you need to do something exotic, just write the code in pure Ruby. 

#### I'm a .NET guy. Isn't Rake just for Ruby projects?

No. With a handy gem called [Albacore](https://github.com/Albacore/albacore), Rake is able to work with .NET tools such as MSBuild, CSC, NUnit and Mono's xbuild. 

#### Getting started

0. Install [Ruby](http://rubyinstaller.org/) or get Ruby for "free" with [Git](http://git-scm.com/download/win)

0. Open up a command line window. I recommend [Console2](http://sourceforge.net/projects/console/) because it supports tabs and multiple shells. I use the version of Bash that comes with Git.

0. From command line, install Albacore using [RubyGems](http://rubygems.org/), a Ruby package manager. 

       gem install albacore

0. Navigate to your solution directory and create your first rakefile.

       touch rakefile.rb

0. Open up your [favorite text editor](http://www.sublimetext.com/) and edit rakefile.rb

        require 'albacore'

        msbuild :build do |b|
            b.properties = { :configuration => :Debug }
            b.targets = [ :Build ]
            b.solution = "HelloWorld.sln"
        end

0. Return to the command line and type

        rake build

Congrats! You just built your first .NET solution using Rake.

#### Woah there, slow down! What just happened?!

If you're note familiar with Ruby, this might be a lot to absorb, especially the syntax. I'll walk through the rake file from a C# perspective. 

First thing we do is include 'albacore'.

A rakefile is a collection of tasks, so next we set up an MSBuild task called `:build`. Rake has a rich [Domain Specific Language](http://en.wikipedia.org/wiki/Domain-specific_language), which allows us to write build scripts more naturally. What we're actually doing here is calling the `msbuild` function in the albacore namespace to register a new msbuild task by giving it a name and an anonymous function.

In case you're wondering what `:build` is, it's a Ruby symbol. The simplified answer is that you can think of them as enums that you can use like strings. In fact, we could have easily used a string instead symbol to name our task:

    msbuild "build" do |b|
      # ...
    end

As I mentioned earlier, `msbuild` is a function that expects an anonymous function as its second parameter. In C#, that would be an Action or Func object. Ruby uses do-end blocks. In our anonymous method we have access to the parameter `b` and we set the necessary properties on it.

In C#, this rakefile would look something like this:

    using Albacore;

    namespace HelloWorldRakefile
    {
        class Program
        {
            static void Main(string[] args)
            {
                var rake = new Rakefile();

                rake.AddTask<MSBuildProps>("build", b => {
                    b.Properties = new Dictionary<string, string> { { "Configuration", "Debug" } };
                    b.Targets = new [] { "Build" };
                    b.Solution = "HelloWorld.sln";
                });

                rake.AddTask("default", "build");

                rake.Run(args);
            }
        }
    }

And you would run it like this:

    HelloWorldRakefile.exe build

#### Let's make this better

Now that you're familiar with Rake, you can make this even better by setting up a default task that runs the `:build` task

    task :default => :build

This allows you to simply run the `rake` command without specifying a task.

Rake also allows for sequencing tasks together. If we had a `:clean` task, we could update :default to call :clean before calling :build. 

    require 'albacore'

    msbuild :build do |b|
        b.properties = { :configuration => :Debug }
        b.targets = [ :Build ]
        b.solution = "HelloWorld.sln"
    end

    msbuild :clean do |b|
        b.properties = { :configuration => :Debug }
        b.targets = [ :Clean ]
        b.solution = "HelloWorld.sln"
    end

    task :default => [ :clean, :build ]

#### To infinity and beyond

I've only scratched the surface... Rake and Albacore can do a lot more. 

Rake isn't simply useful for compiling code and copying files. You can make a Rake task for anything scriptable, like sending a tweet. I often find it easier to create a `rakefile` than to write a ruby or batch script. 

Cheers!
