---
layout: post
title: "Copy command line output to the clipboard on macOS, Windows, and Linux"
author: "Adam Patridge"
date: Fri, 27 Sep 2019 13:30:44 +0000
tags: command-line
# category: dev
excerpt_separator: <!--more-->
---

NOTE: Updated for PowerShell 7, with `Set-Clipboard` available to supported platforms (Linux still requires **xclip**).

You've just figured out the exact piece of information you need from a command line call. Now, how do you get it out of your terminal and take it somewhere else, like an email, chat, or document?

Fortunately, there are already programs out there to make this easy. You can send the output from any command to one of these applications, and the output will be available in your clipboard. From there, you can paste it anywhere you please. In the case of Windows and macOS, there are programs to do this that come with the OS. On Linux, you can install a tool to have the same functionality.

<!--more-->

### Windows

Whether you are in Command Prompt or PowerShell (old-school or Core), you can use the Windows **clip** application to get output to the clipboard. Should it matter to you, clip will append an extra line break to whatever you feed it.

```command
{some command} | clip
```

For example, to dump your file listing to the clipboard, you would run `dir | clip` or `Get-ChildItem . | clip`, with `dir` working in both cmd and all flavors of PowerShell and the later only working in PowerShell.

PowerShell 7 or the older non-Core PowerShell also offers its own command: `Set-Clipboard` that won't append the line break. You can also pass it the `-Append` parameter to keep building a result in the clipboard.

```powershell
{some command} | Set-Clipboard
```

The `Set-Clipboard` command isn't available on PowerShell Core versions before 7. For those PowerShell Core version, you'll have to pass your output to the OS-specific version like `clip` or `pbcopy`, which is the magic program to use on macOS.

### macOS

Whether you are in Terminal or PowerShell Core, you can use the macOS **pbcopy** application to get output to the clipboard. Should it matter to you, pbcopy will append an extra line break to whatever you feed it.

```bash
{some command} | pbcopy
```

So, for example, if you want to dump the current directory's file listing to the clipboard, you would run `ls | pbcopy` or `Get-ChildItem . | pbcopy`.

PowerShell 7 also offers its own command: `Set-Clipboard`. You can also pass it the `-Append` parameter to keep building a result in the clipboard.

```powershell
{some command} | Set-Clipboard
```

### Linux (probably not all variants)

To do the same magic on Linux, you need to install a program to do the work for you. One option is **xclip**. You can install it with `apt-get`.

```bash
sudo apt-get install xclip
```

From there, you can start passing things to the clipboard using xclip as your destination.

```bash
{some command} | xclip
```

PowerShell 7 also offers its own command: `Set-Clipboard`. You can also pass it the `-Append` parameter to keep building a result in the clipboard.

```powershell
{some command} | Set-Clipboard
```
