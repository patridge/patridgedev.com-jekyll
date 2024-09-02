---
layout: post
title: "Mono for Android: &quot;aapt.exe&quot; exited with code 1"
date: Thu, 12 Jul 2012 00:26:32 +0000
tags: Xamarin Xamarin.Android Mono-for-Android C#
# category: dev
excerpt_separator: <!--more-->
---

### TL;DR

Also available in [TL;SO (too long; Stack Overflow) flavor](http://stackoverflow.com/q/11440741/48700).

Getting this error: `"aapt.exe" exited with code 1`?

Do you have any files in your Mono for Android solution that are being packaged together with the app (e.g., "AndroidResource" build action)?

If so, make sure they don't have anything but letters, numbers, periods, and underscores ([a-z0-9_.]) in their names.

<!--more-->

### Details

I am still getting my feet wet with Mono for Android (MfA) development. One of my first projects was a [flashlight app](https://github.com/patridge/MonoForAndroidFlashlight) for my Galaxy Nexus that introduced me to [a](http://stackoverflow.com/q/6881004/48700) [couple](http://stackoverflow.com/q/10757138/48700) Android/MfA development quirks. One concept in app development that I would like to explore is long-running tasks. How do I keep something going after the user has switched off to something else?

Greg Shackles has a new [Visual Studio Magazine article on Background Services in MfA](http://visualstudiomagazine.com/Articles/2012/07/10/Background-Services-in-Mono-for-Android.aspx?Page=1) on just that topic. He creates a background service for playing an MP3 file using a standard Android service.

In his sample project, he sets up the service to play some Creative Commons [Nine Inch Nails music](http://www.nin.com/albums/).

<blockquote class="twitter-tweet"><p>I'm betting that I'm the first person to use (and include in the sample) a Nine Inch Nails song for Visual Studio Magazine. <a href="https://twitter.com/search/%2523winning">#winning</a></p>&mdash; Greg Shackles (@gshackles) <a href="https://twitter.com/gshackles/status/222768944459685890" data-datetime="2012-07-10T19:07:09+00:00">July 10, 2012</a></blockquote>

I happen to have a good collection of my own Nine Inch Nails music. As I put the code together in my own project, I grabbed one such MP3 file, named "02 1,000,000.mp3", and added it with the "AndroidResource" Build Action. Unfortunately, hitting <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>B</kbd> resulted in a lovely error: `"aapt.exe" exited with code 1`.

<a href="/wp-content/uploads/2012/07/BuildError.png"><img src="/wp-content/uploads/2012/07/BuildError-300x127.png" alt="" title="Build Error" width="300" height="127" class="alignnone size-medium wp-image-187" /></a>

If you go to the Visual Studio output window, you don't find much else of value.

<a href="/wp-content/uploads/2012/07/MinimalMsbuildLogOutput.png"><img src="/wp-content/uploads/2012/07/MinimalMsbuildLogOutput-300x165.png" alt="" title="Minimal MsBuild Log Output" width="300" height="165" class="alignnone size-medium wp-image-185" /></a>

Fortunately, you can get more verbose output from MSBuild if you ask for it. In the Visual Studio Options, go to "Projects and Solutions" then "Build and Run". Switch the "MSBuild project build output verbosity" from "Minimal" to "Normal". After I built the project again, I got a little clearer message: `res\raw\02 1,000,000.mp3: Invalid file name: must contain only [a-z0-9_.]`.

<a href="/wp-content/uploads/2012/07/NormalMsbuildLogOutput.png"><img src="/wp-content/uploads/2012/07/NormalMsbuildLogOutput-300x164.png" alt="" title="Normal MsBuild Log Output" width="300" height="164" class="alignnone size-medium wp-image-191" /></a>

A quick name change to something within the `[a-z0-9_.]` range and the project built beautifully. While this new sample project is just a horrible version of the baked-in music application, that definitely isn't the point. It is definitely nice to have some code written about a concept before you find yourself needing it the first time.
