---
layout: post
title: Creating a backup routine in F# using a JSON file
categories: [Programming]
tags: [F#, JSON]
comments: true

---
Over the Christmas holidays, I decided to force myself to learn some basic **F#** code. To get things going, I decided to create a small project that would get my hands dirty rather than spending a lot of time reading about all of the good things **F#** can do. I decided to create a simple project that would help me copy some projects at work from my local drive to the network once a week. Start with the basics and then make it complicated.

The project would have to read a `JSON` file with basic information such as the source folder, include subfolder and the destination folder, and then copy the files to the desired drive. It also had to be run from a command prompt.

This little holiday project—I estimated that it would take no more than a few hours—would lead me to learn the basic “how to’s” of **F#** programming. For example, I would familiarize myself with the key **F#** language components, such as: declaring variables, using loops, conditions, functions and some very interesting syntaxes that I’ve never come across. Also, I anticipated that reading a `JSON` file and working with it in **F#** might be fairly easy, but it turned out to be a bit of a challenge.

I created a project in Visual Studio and installed FSharp.Core through Packager Manage Console. (You can also work with packages using the Manage NuGet Packages dialog box.) For my project I had to include a `newtonsoft.json` package as well.

#### Installing **F#**
1. In Visual Studio, from the Tools menu, select Library Package Manager.
2. Click Package Manager Console.

{% highlight Assembly %}
>PM: Install-Package FSharp.Core 
{% endhighlight %}

In Solution Explorer, you can see the references that Visual Studio has added for the installed library or libraries.

You should now be able to start coding in F#.

**Happy Coding!**


_ _ _

<a class="btn btn-default" href="https://github.com/jjpinto/BackupRoutine">Get Backup Routine on GitHub</a>



