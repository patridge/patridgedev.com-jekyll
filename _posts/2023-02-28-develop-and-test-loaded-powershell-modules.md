---
layout: post
title: "Develop and test loaded PowerShell modules"
author: "Adam Patridge"
date: 2023-02-28 20:52:33 -0600
tags: command-line PowerShell
# category: dev
excerpt_separator: <!--more-->
---

I have a slowly evolving PowerShell script, more of a set of copy-paste commands at this point, that is evolving into a [full-fledged module](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/how-to-write-a-powershell-script-module?view=powershell-7.3). I'm working on this module from my usual code location, but I still wish to be able to test it as if it were a registered module.

There are probably a few ways to do this, but here is how I'm making it work, as well as a bonus method that might be a solution for more serious PowerShell module developers. For the primary approach, we'll make Windows think the PowerShell script file is actually in a proper module location.

<!--more-->

## Setup option

First, find where your current PowerShell is looking for modules via the various paths assembled in `$env:PSModulePath` to pick the one you want to use. For my Windows 11 install of PowerShell 7.3.2, my preferred location is in my user directory at a path that can be represented like this: `${env:USERPROFILE}\Documents\PowerShell\Modules\`. You could also add the module in one of the locations shared between all users.

To be able to load a module, it must exist as a `.psm1` file within an identically named folder within one of those module paths. For my module and preferred path, I was creating a module named `VideoProcessing`, so it will exist as `${env:USERPROFILE}\Documents\PowerShell\Modules\VideoProcessing\VideoProcessing.psm1`. At this location, and most module paths, it will require administrator privilege.

Now, from an administrator PowerShell session, create a symbolic link from the development copy of the file to your module folder in a directory named for the module you are creating. Since I am working on a module I want named VideoProcessing, this will create the required structure as it creates the link. (Note that the `-Force` option will create the parent directory, but will also overwrite any existing link, which could be good if you need to update it later.)

```powershell
New-Item -ItemType SymbolicLink -Path "${env:USERPROFILE}\Documents\PowerShell\Modules\VideoProcessing\VideoProcessing.psm1" -Target "C:\dev\PowerShellExperiments\Video-CmdletRemoveVideoSection.psm1" -Force
```

## Load your module (via symbolic link)

Now, I can load that module for use in my current session from the module folder name.

```powershell
Import-Module VideoProcessing
```

Optionally, if you have the module in a shared user location but only want to install it for yourself, you can limit the install with the: `-Scope=CurrentUser`.

```powershell
Import-Module VideoProcessing -Scope=CurrentUser
```

## Making changes and reloading

As you edit your module in its original location, you will need to force-reload the module to test new changes.

```powershell
Import-Module VideoProcessing -Force
# optionally add `-Scope=CurrentUser`
```

## Cleaning up

To remove the module at a later point:

```powershell
Remove-Module VideoProcessing
```

## Another dev approach

Alternatively, if you were developing several PowerShell modules, you could just add that path to your `${env:PSModulePath}` variable. Since I don't have my modules separate from many non-module repos, I won't be testing this approach.

```powershell
$env:PSModulePath += ";C:\dev\PowerShellModules\"
Import-Module VideoProcessing
```
