---
layout: post
title: "Workaround: Xamarin.Android long paths on Windows"
author: "Adam Patridge"
date: Mon, 20 May 2019 14:16:37 +0000
tags: Xamarin.Android Xamarin.Forms
# category: dev
excerpt_separator: <!--more-->
---

## Quick answer (TL;DR)

Create a symbolic link from your deeply-nested folder to a shorter path using one of the following commands in an Administrator prompt.

### Command Prompt

```
mklink /D {desired-path-location} {actual-path-location}
```

### PowerShell [Core]

```
New-Item -ItemType SymbolicLink -Path {desired-path-location} -Value {actual-path-location}
```

After you create the symbolic link, drag the desired solution into Visual Studio rather than navigating through the link in the Open dialog, which will override to the original path.

<!--more-->

## Background

If you like to keep all your code in a user directory on Windows, there's a very good chance you've cloned a repo or started a new Xamarin.Android project that ran into issues on the first build. Most likely you have run into a path length issue. Sometimes a project is just so nested, even putting the repo in a low-level folder will still have trouble.

If you read the error messages, sometimes they explicitly say there is a path length issue. Sometimes, though, it will be something unusual like this JavaTypeInfo error. I don't know enough of the Android build process to explain the issue here, but it's definitely looking for a file at a path location that is 269 characters long.

> Failed to create JavaTypeInfo for class: Android.Support.V4.View.Accessibility.AccessibilityManagerCompat/IAccessibilityStateChangeListenerImplementor due to System.IO.DirectoryNotFoundException: Could not find a part of the path 'C:\Users\patridge\source\repos\mslearn-create-a-mobile-app-with-xamarin-forms\src\exercise1\final\Phoneword\Phoneword.Android\obj\Debug\81\android\src\mono\android\support\v4\view\accessibility\AccessibilityManagerCompat_AccessibilityStateChangeListenerImplementor.java'.
>    at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
>    at System.IO.FileStream.Init(String path, FileMode mode, FileAccess access, Int32 rights, Boolean useRights, FileShare share, Int32 bufferSize, FileOptions options, SECURITY_ATTRIBUTES secAttrs, String msgPath, Boolean bFromProxy, Boolean useLongPath, Boolean checkHost)
>    at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize)
>    at Xamarin.Android.Tools.Files.CopyIfStreamChanged(Stream stream, String destination)
>    at Xamarin.Android.Tasks.Generator.CreateJavaSources(TaskLoggingHelper log, IEnumerable`1 javaTypes, String outputPath, String applicationJavaClass, Boolean useSharedRuntime, Boolean generateOnCreateOverrides, Boolean hasExportReference)

You can definitely run into this issues in a number of situations. Even the [Xamarin modules we recently launched on Microsoft Learn](https://aka.ms/learn-xamarin) have a path like this: **{descriptive-module-name}/src/exercise#/{start\|final}/{SolutionName}**. That's enough to to get too long for Windows without being in a deeply-nested folder to start.

On Windows, when you start nesting files really deep, you run into a [maximum length of 260 characters](https://docs.microsoft.com/en-us/windows/desktop/FileIO/naming-a-file#maximum-path-length-limitation) (drive letter, like "C:\", plus 256-characters of path from there). While there are ways of [circumventing the path length restriction in newer versions of Windows](https://mspoweruser.com/ntfs-260-character-windows-10/), I haven't been willing to try them yet. (I doubt I still use any 32-bit applications, but if I did, they wouldn't be compatible with the long paths.)

## Fake a shorter path

I used to copy entire repos to a shallower folder location like **C:\dev**, or **C:\d\** when I got desperate. But sometimes I would forget which location I was working in and leave uncommitted Git changes in the wrong location. Ideally, I would keep all my code in a single location but not have those issues. If you also run into Android build issues that are caused by path lengths, we can trick Windows and Visual Studio into thinking our path is shorter without moving the files from our desired location.

The magic here comes from _symbolic links_ or _junctions_ in the Windows file system. These two tools are a bit like a shortcut, but it steps in the way of the path resolution as you use files and folders it points to. Regardless of the approach, this is how we can make a file at **C:\Users\awesomeuser\source\repos\AndroidProjectOfAwesomeness** (for Visual Studio 2019; **C:\users\awesomeuser\Documents\Visual Studio 2017\Project\AndroidProjectOfAwesomeness**) appear like it's being used from **C:\dev\Awesomeness**.

> The [difference a symbolic link and a junction in Windows is subtle](https://superuser.com/a/1291446/3675), and really only seems to become an issue when dealing with remote connections. Just know that if you are linking _from_ a local path and _to_ a local path, you can probably use either one without seeing any differences.

## Create a symbolic link in Windows

Enter the `mklink` Windows console command. It's been around for a while, and there was a similar `junction` command in Windows versions before Windows 8, but it doesn't get much attention. Open a command or PowerShell prompt from the Start Menu. (Creating a symbolic link requires running this prompt as an administrator unless you have Developer Mode enabled.) Run a command like the following to link from `{desired-path-location}` (the shortened path) to a location where the code currently exists in `{actual-path-location}` (the longer path where the Android project isn't building).

For the Windows Command Prompt, the command looks like this:

```
mklink /D {desired-path-location} {actual-path-location}
```

For PowerShell, the command looks like this:

```
New-Item -ItemType SymbolicLink -Path {desired-path-location} -Value {actual-path-location}
```

So, if I cloned the intro Xamarin.Forms module exercises from Microsoft Learn to the default location, but I wanted to make sure my path wasn't too crazy, I could run a command similar to this, where "someuser" is your Windows username.

```
mklink /D C:\dev\learn-forms C:\Users\someuser\Documents\source\repos\mslearn-create-a-mobile-app-with-xamarin-forms\src\
```

Or, if you are using PowerShell, the right parameters to `New-Item` will create a symbolic link as well.

```
New-Item -ItemType SymbolicLink -Path C:\dev\learn-forms -Value C:\Users\someuser\Documents\source\repos\mslearn-create-a-mobile-app-with-xamarin-forms\src\
```

From there, I would have access to all the exercise folders, such as the final exercise at **C:\dev\learn-forms\exercise1\final\Phoneword**.

## Using the symbolic link in Visual Studio

For some reason, Visual Studio won't just play along with the illusion of symbolic links when you navigate through one in the normal open dialog. It will navigate to the original path location, defeating the whole point of all this.

However, if you find the solution file you want in Explorer first and drag it into Visual Studio, it will play along just fine. At this point, you should be happily building your Android projects without issue!

## When you're done with a symbolic link

If a symbolic link ever outlives its usefulness, just delete it like it's any other shortcut. The original directory will stay where it is. The flip side is also true, tough. If you delete the source directory, the link won't know any better.

## Bonus tip

This can also be a good way to move location-required files from a drive that is running low on space to an external location. Some notable examples include migrating your iTunes music to a different drive, or moving your installed Steam games to another drive. You create a symbolic link from where iTunes or Steam is looking for those file to where you are planning to store them. The app doesn't know any different, but your C drive can be much less bloated. If you are running Windows from a must-faster-but-smaller SSD, this can be a life saver.
