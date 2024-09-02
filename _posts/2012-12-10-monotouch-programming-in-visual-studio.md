---
layout: post
title: "MonoTouch Programming in Visual Studio"
date: Mon, 10 Dec 2012 20:19:57 +0000
tags: Xamarin Visual-Studio MonoTouch
# category: dev
excerpt_separator: <!--more-->
---

## TL;DR

Never underestimate the little time sinks of switching between IDEs regularly. To write MonoTouch code in Visual Studio 2010 (debug/deploy still requires MonoDevelop on a Mac), go get [VSMonoTouch](https://github.com/follesoe/VSMonoTouch). If you have any issues getting it going, you may need to toss in some project file tweaks.

* [Set it to not reference mscorlib.dll](https://github.com/follesoe/VSMonoTouch/issues/12)
* Set the `System.Web.Services` reference to version 2.0.5.0 (and likely any others that may conflict with the latest .NET runtime assemblies).

<!--more-->

## Background

I've been programming with MonoTouch for a few months now using MonoDevelop. I really enjoy learning new things (even if MonoTouch saved me from learning Objective-C), but switching IDEs always tosses a few kinks in my productivity. I have tweaked a number of key bindings in MonoDevelop to match Visual Studio, some at the cost of my ability to adapt to the Mac's defaults that are normally used everywhere. At one point I switched copy and paste to use Ctrl in MonoDevelop, but then I kept screwing up outside of the IDE. Regardless, I have become fairly productive in MonoDevelop from simply adapting to the new system through repetition (often screwing up when I switch back to Windows now).

I don't really have a problem with MonoDevelop most days, but I definitely prefer the Windows system of maneuvering windows and working on multiple displays. Even if I were a Mac-only person, I would hate how full-screen mode works on a Mac. While I have adapted to my new MonoDevelop workflow, I definitely miss having multiple tabs open in a single project; to open multiple instances of MonoDevelop on a Mac, you even need [a simple hack](http://stackoverflow.com/a/6081311/48700). Having to flip around between MonoDevelop and Chrome as I drink from the MonoTouch learning firehose only compounds the window management issue.

Nothing seems to get you around the need for a Mac when testing/debugging/deploying a MonoTouch project, but having Visual Studio as your primary IDE for all your .NET projects can definitely speed things up.

## VSMonoTouch

Enter [VSMonoTouch](https://github.com/follesoe/VSMonoTouch) by Jonas Follesø ([follesoe on GitHub](https://github.com/follesoe)).

This Visual Studio VSIX add-in (warning: requires a non-free Visual Studio) seems to create a system for allowing Visual Studio to recognize the MonoTouch framework and its project type and use the correct DLLs for compiling. For simplicity's sake, just follow the instructions in the project README for your first run. If that doesn't work, start looking for tweaks.

The final process I used was simply moved the VSIX install step from first to last on the instructions list from the README (with absolutely no reason to explain why it worked). Regardless, feel free to try some restarts on the off chance you have issues.

1. Copy the MonoTouch binaries from your Mac development environment to your Visual Studio 2010 development environment. Copy all the files from /Developer/MonoTouch/usr/lib/mono/2.1/ on your Mac to C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v1.0 on your PC.
1. Add a RedistList-folder under your newly created v1.0-folder. Download the FrameworkList.xml file and add it to the RedistList-folder.
1. Download and run the vsix-package from the github page.

### Code Reference Quirks

It seems that some systems have an easier time getting VSMonoTouch to run than others. On my Windows 7 machine with Visual Studio 2010 (10.0.40219.1 SP1Rel), .NET 4.5, and various Visual Studio 2012 free components; I had a few issues getting things just right. The first three times I tried things out, I simply couldn't get Visual Studio to acknowledge the project type. It would load the solution I have from MonoDevelop but leave the projects unloaded.

I tried all sorts of variations on the FrameworkList.xml file and gave up for a while.

#### mscorlib

Today, I decided to try again after I noticed a few updates to [a project issue with mscorlib](https://github.com/follesoe/VSMonoTouch/issues/12). That wasn't the issue I was having, but I was hoping anyone trying to solve this issue may accidentally find a solution to help my problem. Oddly, after installing the DLLs first and then the VSIX, it all started loading correctly but with some compile errors and warnings.

* Error: "An assembly with the same identity 'mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089' has already been imported. Try removing one of the duplicate references."
* Warning: "No way to resolve conflict between "mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" and "mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089". Choosing "mscorlib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" arbitrarily."
* Warning: "There was a mismatch between the processor architecture of the project being built "MSIL" and the processor architecture of the reference "C:\Windows\Microsoft.NET\Framework\v2.0.50727\mscorlib.dll", "x86". This mismatch may cause runtime failures. Please consider changing the targeted processor architecture of your project through the Configuration Manager so as to align the processor architectures between your project and references, or take a dependency on references with a processor architecture that matches the targeted processor architecture of your project."
* Warning: "Found conflicts between different versions of the same dependent assembly."

This is where that [mscorlib project issue] came into play. There was a note on the VSMonoTouch README about making sure to have explicit mscorlib references, but I already did. Checking the box on the "Do not reference mscorlib.dll" project property, though, removed those errors.

<a href="/wp-content/uploads/2012/12/DoNotReferenceMscorlibDll.png"><img src="/wp-content/uploads/2012/12/DoNotReferenceMscorlibDll-150x150.png" alt="VSMonoTouch mscorlib.dll issue fix" title="DoNotReferenceMscorlibDll" width="150" height="150" class="aligncenter size-thumbnail wp-image-230" /></a>

If you are a fan of tweaking csproj XML directly instead of through the project properties editor, feel free to add this node to any compilation `PropertyGroup`s manually.

    <NoStdLib>true</NoStdLib>

#### System.Web.Services

After fixing the mscorlib.dll issues, though, I got a new error popping up. This final compile error was coming from code I wrote, so that was a good sign (I guess).

* Error: "The name 'HttpUtility' does not exist in the current context"

Since I am writing an application that interfaces almost entirely with the [web-based JSON API](https://dev.sierratradingpost.com/) I wrote, there are a few places where I am sending back user content to that system. The URL encoding I use simply hits `HttpUtility.UrlEncode`. In MonoTouch, you need to make sure you [reference `System.Web.Services` to get `UrlEncode`](http://stackoverflow.com/a/5398253/48700). But, in Visual Studio Land, the .NET 4.0+ DLL doesn't have such a method. You can fix this issue with a static version reference on that DLL. There may be a version-agnostic way to do this, but I ended up editing my csproj file to point to the specific version found in the MonoTouch DLLs.

    <Reference Include="System.Web.Services, Version=2.0.5.0" />

I will probably have to update this manually whenever a new version hits MonoTouch, but I definitely can live with that.

### VSMonoTouch In Use

After getting the loading and compile errors worked out, I now have a MonoTouch solution building successfully in Visual Studio 2010. It contains a core project shared with the [Sierra Trading Post Android app](https://play.google.com/store/apps/details?id=com.sierratradingpost.android) (using a few compiler directives) and my MonoTouch project containing almost entirely UI-related code. I write some code and it tells me whether it will compile correctly against the MonoTouch DLLs. I haven't tried abusing the reliability of this system with any features that are not part of Mono (e.g., async/await) or MonoTouch (e.g., dynamic) yet.

Since my project is currently synced through Dropbox, I just save what I have on Windows and roll on over to the MacBook to run things on the simulator or deploy them to a physical device and/or [TestFlight](https://www.testflightapp.com/) (which is far better than sliced bread, at least for getting your app out to multiple devices). MonoDevelop will even automatically update your project files if you already have it open, though I don't recommend editing on both systems simultaneously without having something between them to handle conflicts that are bound to arise.
