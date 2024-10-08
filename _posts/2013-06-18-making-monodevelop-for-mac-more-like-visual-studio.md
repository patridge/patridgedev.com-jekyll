---
layout: post
title: "Making MonoDevelop for Mac (or Xamarin Studio) more like Visual Studio"
author: "Adam Patridge"
date: Tue, 18 Jun 2013 15:19:25 +0000
tags: Xamarin Visual-Studio MonoDevelop
category: dev
excerpt_separator: <!--more-->
---

If I were only living in the land of MonoDevelop, I probably wouldn't care. I stil spend a large amount of time using Visual Studio, though, and switching contexts becomes very difficult when the IDEs are so different. While I haven't gone as far as to swap the command and alt keys to match keyboard behavior on Windows, I do try to unite things as much as possible.

If you are simply looking for a list of the default MonoDevelop keyboard shortcuts to learn, check out something more like [this post from Dan Quirk](http://maccork.com/2011/02/26/monodevelop-keyboard-shortcuts/).

<!--more-->

[Note: Most of these things apply to Xamarin Studio as well.]

## Text editing

### Word break mode

This made the biggest difference in my tolerance for MonoDevelop. The default word break mode tends to ignore whitespace, making it behave completely differently moving the cursor forward versus backward in text.

*Preferences->Text Editor->Behavior; Navigation->Word break mode; set to SharpDevelop.*

This will bring your option+left/right and option+backspace/delete calls to be more in line with the ctrl versions under Visual Studio. The emacs version looks like it would work, but I opted for SharpDevelop.

## Key bindings

### <kbd>ctrl</kbd>+<kbd>Space</kbd>

Bring up contextual menu for Intellisense (`Edit.CompleteWord` in VS, "Show Completion Window" in MD).

### <kbd>ctrl</kbd>+<kbd>.</kbd>

Bring up contextual menu to resolve with `using` statement (`View.ShowSmartTag` in VS, "Quick fix..." in MD).

VS: <kbd>ctrl</kbd>+<kbd>.</kbd>
MD: <kbd>option</kbd>+<kbd>enter</kbd> (does not appear to work)

#Changes:#

Since the default MD shortcut doesn't work, you definitely want to try something. Setting this to the VS default, though, will cause the following cascade of changes.

* <kbd>Command</kbd>+<kbd>.</kbd> is already "Navigate To..." in MD (VS uses <kbd>ctrl</kbd>+<kbd>,</kbd>).
* <kbd>Command</kbd>+<kbd>,</kbd> is already "Preferences..." in MD (VS default doesn't bind anything to `Tools.Options` but <kbd>command</kbd>+<kbd>,</kbd>, pretty standard Mac shortcut).
* Remove <kbd>command</kbd>+<kbd>,</kbd> from "Preferences...".

## Export/Import settings

Once you have MonoDevelop just the way you want it, some day you will likely need to move them to a different machine (or give them to a like-minded colleague). You'll need to dig around your user library to find these, but they can be copied into and out of `~/Library/MonoDevelop-3.0` (or whatever version you are using). Xamarin Studio will be under `~/Library/XamarinStudio-{version}`.

For just specific preferences, you may be able to move around just a single file.

* Key bindings: `~/Library/MonoDevelop-3.0/KeyBindings/Custom.mac-kb.xml`
* C# formatting profile: `~/Library/MonoDevelop-3.0/Policies/Default.mdpolicy.xml`

### Sources

* Word break mode tip via [Michael J Hutchinson](http://mjhutchinson.com/journal/2011/02/monodevelop_tips_word_breaking).
* Settings export tip via [Eugen Rieck's Stack Overflow answer](http://stackoverflow.com/q/9313350/48700).

### Versions
Since these things are likely to change, let me add the versions of these applications. If things have become too dated here, feel free to drop me a line (comments, email, whatever).

* Mac OS: 10.7.5
* MonoDevelop: 3.0.4.7
* MonoTouch: 3.0.4

---

## Comments

* **Victor Chelaru**, _2015-10-14 20:30:58 +0000_:

    > Turns out that the Libraries folder is hidden under the user's root in OSX 10. To solve this:
    >
    > http://osxdaily.com/2011/07/04/show-library-directory-in-mac-os-x-lion/
    >
    > Specifically:
    >
    > Open a terminal and enter/run: `chflags nohidden ~/Library/`
    >
    > Once you do this, you should be able to spot the XamarinStudio folder as mentioned in the above post.
