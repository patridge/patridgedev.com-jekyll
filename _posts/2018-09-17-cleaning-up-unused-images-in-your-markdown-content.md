---
layout: post
title: "Cleaning up unused images in your Markdown content with PowerShell"
author: "Adam Patridge"
date: Mon, 17 Sep 2018 14:00:06 +0000
tags: command-line PowerShell
category: dev
excerpt_separator: <!--more-->
---

I was recently tasked with cleaning up some Markdown content with a bunch of screenshots. Sometimes as content was revised, an image would no longer be used, but the image wasn't deleted. As a result, the images folder would often be packed with files that were no longer used in the final Markdown content.

On a few blocks of content, I would do this manually in VS Code. From the file list (Ctrl+Shift+E), I'd select the file, copy the file name (F2, then Ctrl+C), search all the files for that file name (Ctrl+Shift+F, then Ctrl+V). This was painful to do for more than a few blocks, so I decided to turn to automation, Powershell in this case.

PowerShell is available on Windows and Linux/macOS, so it's great for wherever I need it. It even seems to properly translate my path separators on different platforms.

<!--more-->

## Getting the image file names

There are a lot of aliases in PowerShell, they make some verbose commands either shorter and easier to remember, or they duplicate functionality found in the host system. For example, the `dir` command is available in PowerShell, but it is actually an alias for the `Get-ChildItem` command. (Please note that PowerShell commands are actually called "cmdlets", just not here…sorry, not sorry.)

So, to get all the image file names I want to search for, I'll need a variant of `Get-ChildItem`. In my case, these images were in a "media" folder.

<!-- language: powershell -->

    Get-ChildItem .\media\

This will list out my files, but let's map the resulting objects into just the filename strings.

<!-- language: powershell -->

    Get-ChildItem .\media\ | ForEach { $_.Name }

Now, we "pipe" (`|`) the resulting `Get-ChildItem` file objects individually into the `ForEach-Object` command (or just `ForEach` as we used), and for each item, noted with `$_`, we grab the `Name` property to make sure we have just the filename and not the full file path.

## Searching a file for a string

As a quick side-step, let's figure out how to search a file for a particular string. In PowerShell, there is the [`Select-String`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string?view=powershell-6) command, which the docs say, "Finds text in strings and files."  It can take a string and return the line containing the given `-Pattern` parameter. I won't argue it's the fastest or most accurate way to do text search in files, as I haven't done any research on alternatives, but it works well enough for my use case.

<!-- language: powershell -->

    Select-String -Pattern "something awesome" -Path .\content\*.md

I'm not stressing this command too much. Select-String can do much more advanced stuff, especially if you need regular expressions.

## Combination: Find unused images

With a list of image file names, I can now use another command to filter the list: `Where-Object` (or just `Where`). If you are familiar with C#, this is just like the `Where` function in LINQ. WIth any collection, you pass in a method that returns a boolean object. Anything that would return true on that method will make it into the resulting output.

In this case, for each image file name, I want to see if it **doesn't** produce any results when run through `Select-String` on any of the Markdown files.

<!-- language: powershell -->

    Get-ChildItem .\media\ | Where { (Get-ChildItem .\includes\*.md | Select-String $_.Name).Count -eq 0 }

Things start to get a little nested here. We look for the name property of each image file within all the Markdown files (those with the `.md` extension), and only leave the ones with no matches (`.Count -eq 0`) to each one of the file names (`$_` of the `Where` method). Any image file name that makes it through was not mentioned in the Markdown content, which in my case, means it is fair game for deletion.

## Deleting a file

With one last side-step, let's figure out how to delete a file. In this situation, any image I have that isn't referenced is bound for the trash.

<!-- language: powershell -->

    Remove-Item -Path .\media\some-image.png

The `Remove-Item` command will delete the item at the given path. That path can be a pattern or even a collection, which will be handy when I have a list of images I no longer want. 

## Final combination: Deleting images unused in Markdown

With all the commands and a little bit of piping them together, I can now assemble a command that will find all unused images and delete them immediately. In this case, we take the output of finding unused images, filter to just the fully-qualified path (`$_.FullName`) and pipe it directly into the `Remove-Item` command.

<!-- language: powershell -->

    Get-ChildItem .\media\ | Where { (Get-ChildItem .\includes\*.md | Select-String $_.Name).Count -eq 0 } | ForEach { $_.FullName } | Remove-Item

And with that, goodbye unused images.
