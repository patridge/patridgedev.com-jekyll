---
layout: post
title: "Adapting Visual Studio code styling differences for open source project contribution"
date: Thu, 01 Dec 2011 14:38:59 +0000
tags: code-formatting open-source-contribution Visual-Studio
# category: dev
excerpt_separator: <!--more-->
---

## Background

Today, while incorporating [Lee Dumond's](https://twitter.com/#!/LeeDumond) [MVC Route/URL Generation Unit Tester](http://mvcrouteunittester.codeplex.com/) into a project, I found a desire to contribute some code I thought would make the package easier to use. Unfortunately, the project code formatting looks nothing like my preferred conventions (some form of [1TBS](http://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS), I guess). Until Visual Studio offers a way to distribute code style settings to source control consumers easily, I needed a different option.

While preparing demos for a mobile web development talk for the Cheyenne Computer Professionals group, I stumbled on [Mike Minutillo's](http://twitter.com/wolfbyte) tip for [running a "demo" instance of Visual Studio](http://codermike.com/vs2010-demo-instance) where I could sandbox my presentation settings optimized for an elderly VGA projector. This sparked an workaround idea for dealing with multiple formatting settings of various projects I may work on.

Rather than force my conventions on the project (generally not acceptable) or give up on my own style (generally not acceptable), I decided to try using a "demo" instance of Visual Studio with that projects styling conventions set.

<!--more-->

1. Right-click where I want the shortcut.
1. Specify a path to Visual Studio 2010 using the `/RootSuffix` option.
  1. On 64-bit Windows, `%programfiles(x86)%\Microsoft Visual Studio 10.0\Common7\IDE\devenv.exe"
 /RootSuffix YourStylingNameHere`
  2. On 32-bit Windows, `%programfiles%\Microsoft Visual Studio 10.0\Common7\IDE\devenv.exe" /RootSuffix YourStylingNameHere`
1. Give the shortcut a name. I tend to go something like this: "Visual Studio (SomeProjectName)".

Using this method, I can create a few [common formatting variations](http://en.wikipedia.org/wiki/Indent_style). Then, whenever I want to work on a given project, I will just slap a new Visual Studio shortcut pointing to the appropriate suffix matching that projects formatting style.

I created an instance of Visual Studio that uses the default settings. Any new project I want to play with that uses those settings gets a custom VS shortcut that points to my "DefaultVsFormatting" root suffix.

By keeping the suffix names consolidated, all projects using those settings are present in that instance's recent projects, both in the File menu and the Start Page (and its pinned projects).

## Drawbacks

While it is great to be able to spawn up a Visual Studio instance, it does complicate some things. If I have multiple instances of Visual Studio running with different prefixes, I have no way of telling which is which without knowing what project I opened from what prefix instance.

It also presents a problem for someone who develops on multiple machines. I want the RootSuffixes I use to be identical on all machines. For the default settings, assuming I have Visual Studio in the same location on every machine, I could simply keep these `%programfiles(x86)%`-based shortcuts on a shared location like my [Dropbox](http://db.tt/MM5CStl) folder where they would work for all machines. If I have custom settings, though, I would need to keep copies of the settings folder data and registry settings.

Keeping the file system part consistent seems easy enough; I could just use a symbolic link to point Visual Studio to them in their shared location (change to your own path to settings folders).

    mklink /d %LocalAppData%\Microsoft\VisualStudio\10.0DefaultVsFormatting %userprofile%\Dropbox\VisualStudioSettings\DefaultVsFormatting
    
There are also registry keys that are part of this process, too. Unfortunately, it appears that all the formatting settings are maintained there. This would take some extra work and make any changes difficult to propagate. After I create a suffix variant, I need to export the registry settings for the two registry keys (folders in the tree).

1. Run 'regedit'.
2. Export the two prefix-based keys that are created ("10.0DefaultVsFOrmatting" and "10.0DefaultVsFOrmatting_Config", in this case).
3. Import those two keys on any other machines.

From there, any time I make a config change, I need to repeat this process on any machine that shares these settings. Not ideal, but it gets the job done.

*UPDATE:*

Unfortunately, after trying this, I found out many of the registry settings are set to explicit user-based paths. I need to try making those paths relative to various path variables (e.g., `%userprofile%`) before exporting the registry key and importing it elsewhere, but that may blow up Visual Studio. If you use the same user on all of your development machines, you are likely fine with this process; if not, you will need to tweak them accordingly.

## Alternative (Visual Studio settings files)

[Jon Galloway's](http://weblogs.asp.net/jgalloway/) [comment](http://codermike.com/vs2010-demo-instance#comment-223) on Mike's post has a valid point. It is definitely a useful option to just use a [custom Visual Studio settings file](http://blogs.msdn.com/b/pstubbs/archive/2010/06/16/presentation-mode-in-visual-studio.aspx)  (via [Paul Stubbs](http://social.msdn.microsoft.com/profile/paul%20stubbs%20%5Bmsft%5D/)) that can be imported as desired. I ended up using the `/RootSuffix` route for a couple reasons:

* Every time I make a common change to the settings (admittedly rare), I would need to update all my various settings files.
* Importing settings files would take more steps than launching a shortcut, at least unless there is some command-line option I haven't learned yet where I can use to give Visual Studio the settings for that run.

No matter what method I use, it gets even more difficult to manage these situations when someone has a tool like JustCode or ReSharper installed on their machine. They would then need to maintain settings files for these tools as well, potentially needing to uninstall them for a given project to keep them from going all Rambo on the foreign code.

## Conclusion

Ideally, any given Visual Studio-based project would just have a file shared among contributors that enforces the code styling conventions chosen for the project. 

Until settings are a part of a project this solution works great on a small scale but is a bit difficult to scale to multiple machines. That said, I only have two machines on which I actively write code, so I will live with the hassle for now.
