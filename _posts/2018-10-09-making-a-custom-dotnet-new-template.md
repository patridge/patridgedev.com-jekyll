---
layout: post
title: "Making a custom `dotnet new` template"
date: Tue, 09 Oct 2018 13:00:23 +0000
tags: dotnet template
# category: dev
excerpt_separator: <!--more-->
---

## Why make a template

I am a bit biased these days, but once I spin up a folder structure and/or text document manually more than twice, I give some thought to templating it. If I need to keep making more of something, the time to get all the boilerplate content in place is time taken away from the good stuff I want to write.

> This is the first in a series of posts about [creating custom templates for the `dotnet new` system](https://www.patridgedev.com/tag/template/).

<!--more-->

## How to template

There are lots of templating systems out there that can allow you to create content quickly. If you have used any of the ASP.NET generators, those use Yeoman to spin up new ASP.NET projects quickly. I'm not saying any template engine is better here, just showing what I've been using lately.

I have been using the templating engine built into the .NET Core SDK, since it tends to be installed on all my development machines. The .NET Core template system is exposed through the `dotnet new` command-line program, where the `dotnet` program is able to do way more stuff like compiling code, managing NuGet packages, or running .NET Core applications. (Run `dotnet --help` to see a list of all the SDK commands available to you.)

That said, if you need to be able to create content from templates in an environment where you don't want the full .NET Core SDK installed, you may want to consider other alternatives, or research how to put just a core templating engine into your own tools and deliverables.

## Get started with .NET Core SDK

You can get the .NET Core SDK installed on macOS, Windows, and Linux. I have it on my MacBook, a couple Windows 10 machines, and a Raspberry Pi or two. If you work with Visual Studio 2017 or Visual Studio for Mac, you may have it installed already, too. From your favorite command line, run the following command to get the current installed .NET Core SDK version on your machine.

<!-- language: bash -->

    dotnet --version

If that fails for you, you can still join in the fun. Run over to the [.NET getting-started guide](https://www.microsoft.com/net/learn/get-started/) and click the download link. That link should land you in the right spot for your current platform. After you install the SDK, you may need to close and relaunch any command line windows you had open previously.

Once you have the .NET SDK installed, give that command another run to make sure everything is happy. I am running version `2.1.300`, which is the [latest release](https://github.com/dotnet/sdk/releases) as of writing this. If you got something like `2.1.###`, you _should_ be able to do everything I'm doing here (taken with a big Works on My Machine&#x2122; disclaimer).

## Create a project

The default .NET Core SDK installation will give you some initial templates if you want to give them a shot. To list all the available templates, just run the following command with no parameters.

<!-- language: bash -->

    dotnet new --list

You should have at least some .NET Core templates for console apps, test projects, and ASP.NET projects. Let's spin up a .NET Core console app. Each of the templates has a "short name" you can pass to `dotnet new`. If you use a template without any parameters, it will put all the resulting files in your current directory. In this case, this command will create a C# project file named after the current directory and code file (as well as some results in **/obj** because it also does a NuGet restore).

<!-- language: bash -->

    dotnet new console

Don't worry, though. If you run this command in a directory where you have conflicting files, it won't overwrite them. You'll get an error for any conflicts.

So you don't have to worry about polluting your current directory, you can give the command another option to drop the template output into a new destination.

<!-- language: bash -->

    dotnet new console --output MyConsoleApp

With the `--output` parameter and value, the result will be put in the **/MyConsoleApp** folder.

While many of these templates create projects you can immediately compile and/or run, that certainly isn't a requirement. In this case, you could run the following commands to execute the resulting console project.

<!-- language: bash -->

    cd MyConsoleApp
    dotnet run MyConsoleApp.csproj

## I want my own template

Now we can start having fun. Let's say you need to make your own console template. The only difference between the console project we generated and a template of our own is a **.template.config** folder containing a **template.json** file. First, make a change to your **MyConsoleApp** project to make it your own. A simple-yet-noticeable change would be to edit **Program.cs** to output your own phrase. In my case, I changed the `Console.WriteLine` call to this.

<!-- language: bash -->

    Console.WriteLine("Hello from a new template!");

Now, we just need the template configuration.

Create a new folder in **MyConsoleApp** called **.template.config**.

Open your favorite text editor and create a new file. Save it to the **.template.config** folder as **template.json**. This JSON file is just the configuration for our template. Here is a sample of the bare minimum this file we might use for our console app template.

<!-- language: json -->

    {
        "$schema": "http://json.schemastore.org/template",
        "author": "Adam Patridge",
        "classifications": [ "Console" ],
        "identity": "PatridgeDev.TemplateBlogging.Console.CSharp",
        "name": "Our slightly modified console app",
        "shortName": "console-awesome"
    }

The `$schema` value is always the same. This just tells compatible IDEs how to offer assistance when creating these documents. For example, Visual Studio Code will use the schema to offer some IntelliSense to us.

The `author` value is you.

The `classifications` value is an array of descriptions you might use for your project. Not going to lie, while these are listed when you do `dotnet new --list`, I have yet to figure out how to query them, or if you even would before you have dozens of templates.

The `identity` value is just a unique name for our template within the `dotnet new` list. You can use some sort of reverse-DNS notation as I did or just about anything. (There seems to be a trend to put language in the last portion, but that is likely only needed if you have multiple language versions of a single template.)

The `name` value is the user-visible name for your template.

Lastly, the `shortName` value is the name people can use when creating things from templates via command line, in the form of `dotnet new {shortName}`.

Customize the JSON fields to your liking and save the result. With that file in place, we can load our template into the .NET Core SDK with a single command.

<!-- language: bash -->

    dotnet new --install .

This command will install the current directory (`.`) as a template. If your command line's current directory is somewhere other than the template, adjust accordingly.

Once you have your template installed, you should see it show up in the list with `dotnet new --list`. (It also should have output the list after successfully installing the template.)

<!-- language: output -->

    Templates                                                   	Short Name       	Language      	Tags
    --------------------------------------------------------------------------------------------------------------------------
    Console Application                                         	console          	[C#], F#, VB  	Common/Console
    …
    Our slightly modified console app                           	console-awesome                    	Console

With our custom template in the list, we can generate a new version of our modified console app now.

<!-- language: bash -->

    dotnet new console-awesome --output ../MyNewAwesomeConsoleApp

This will generate a console app from our template in a sibling folder to our template folder. If you wish to generate the console app elsewhere, adjust the path provided to the `--output` parameter.

Look for your modified line(s) to verify everything generated as expected.

Technically, we have taken a fairly complex template we know nothing about internally and turned it into a completely static template. That's progress, right? We'll work on making our templates customizable another time.

## Other installation options

The `dotnet new --install` command can accept a path to a folder, as we are providing here. It can also accept a path to a local NuGet package (.nupkg) or even a NuGet package ID from nuget.org. For example, I’m working on some templates to help with creating new template and adding files to Git/GitHub repos. This command would install those templates from NuGet directly.

<!-- language: bash -->

    dotnet new --install dotnet_new_template_creator

Once you have a template you want to share with folks, you can wrap it up in a nupkg file and send it to them directly or via publishing it to nuget.org.

## [optional] Uninstalling your template

I won't pretend this new custom template is incredibly useful, so let's make sure we can clean up our test efforts from the `dotnet new` options. If you install a template from a folder location, you uninstall it from an absolute path to that folder location using `dotnet new --uninstall`. For example, if I was working at `~/Projects/`, my uninstall command would be the absolute form of that path and look like this, with your username instead of `{username}`.

<!-- language: bash -->

    dotnet new --uninstall "/Users/{username}/Projects/MyConsoleApp"

If you installed a template from a nupkg or NuGet directly, you pass in the package ID. And, if you are ever unsure how a template was installed, you can get a list of all the valid uninstall parameters with an empty `--uninstall` command.

<!-- language: bash -->

    dotnet new --uninstall

## Check your work

If you got lost along the way or need something to compare to, look in the [**1-custom-template** folder](https://github.com/patridge/demo-custom-dotnet-template/tree/master/1-custom-template) of the [sample Git repo with the resulting templates](https://github.com/patridge/demo-custom-dotnet-template) from following along with this blog post series.

## Next steps

That’s the basics to getting started with your own `dotnet new` templates, but there are so many parts of the templating system that could have their own blog posts. In later posts, I’m hoping to explore some of the more complex features of the template config. If there’s something cool that I need to cover, [reach out to me on Twitter: @patridgedev](https://twitter.com/patridgedev).
