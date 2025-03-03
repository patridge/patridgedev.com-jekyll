---
layout: post
title: "Closing and Re-opening tabs in Visual Studio with Ctrl+W"
author: "Adam Patridge"
date: Tue, 14 Feb 2012 16:13:17 +0000
tags: keyboard-shortcut visual-studio
category: dev
excerpt_separator: <!--more-->
---

**Visual Studio 2017 Undo-Close Update: The Productivity Power Tools have spun off into a bunch of [more-focused extensions](https://github.com/Microsoft/VS-PPT#productivity-power-tools-vs2017). To get Undo Close in Visual Studio 2017, you will want the [Power Commands extension](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.PowerCommandsforVisualStudio) now.**

**Visual Studio 2013 Undo-Close Update: Since the prior options for re-opening closed tabs fell apart with the release of Visual Studio 2013, you will need the newly released [Productivity Power Tools 2013](http://visualstudiogallery.msdn.microsoft.com/dbcb8670-889e-4a54-a226-a48a15e4cace?SRC=VSIDE).**

**Update: now with the ability to [re-open closed tabs](#closing-tabs-in-visual-studio-with-ctrl-w-re-open-closed-tabs) with <kbd>Ctrl+Shift+T</kbd>. This also allows you to re-open tabs closed by a project file reload, which is fantastic!**

Ever tried to close a tab in Visual Studio 2010/2012 with <kbd>Ctrl+W</kbd>. If so, you find yourself selecting the current word in your text editor (Edit.SelectCurrentWord). I don't use that shortcut, though I could see it being handy over my usual <kbd>Ctrl+Shift+Right-/Left-Arrow</kbd>. I do, however, use Ctrl+W to close windows/tabs in just about every other program I use. In order to make that shortcut work for your Visual Studio editing, you just need to assign it to File.Close instead.

<!--more-->

For the visual, here's a snapshot similar to what you will want (note: I already made this change before snapping a pic, so yours may look slightly different).

[![Visual Studio Keyboard Options](/wp-content/uploads/2012/02/Settings-300x174.png "Getting Ctrl+W to trigger File.Close.")](/wp-content/uploads/2012/02/Settings.png)

1. Go into the Tools->Options menu.
1. Select Environment->Keyboard in that window.
1. Type "File.Close" in the "Show commands containing" field and select it in the list when it shows.
1. Choose "Text Editor" for the "Use new shortcut in" field.
1. In "Press shortcut keys" field, press <kbd>Ctrl+W</kbd>.
1. Click "Assign" to make it happen.
1. Close the Options window with the "OK" button.

Now, you can happily close windows with Ctrl+W in Visual Studio.

### <del>Next step</del>

<del>Figure out how to get <kbd>Ctrl+Shift+T</kbd> to bring back the last closed window. If I find a way, I'll post it.</del>

### Update (2012-02-16): Re-open previously closed tabs (with PowerCommands for Visual Studio 2010 [also works for Visual Studio 2012]) {#closing-tabs-in-visual-studio-with-ctrl-w-re-open-closed-tabs}

As [Sergey Vlasov](http://www.svprogramming.net/) [pointed out](/2012/02/14/closing-tabs-in-visual-studio-with-ctrlw/#comment-126) in a comment, the [PowerCommands for Visual Studio 2010](http://visualstudiogallery.msdn.microsoft.com/e5f41ad9-4edc-4912-bca3-91147db95b99/) adds the ability to "Undo Close" on previously closed tabs. Since it adds them as commands to Visual Studio, you can also override the default keyboard shortcut (<kbd>Ctrl+Shift+Z</kbd>) to a more web-browser-esque shortcut: <kbd>Ctrl+Shift+T</kbd>. If you already had this installed for Visual Studio 2010 when you started using Visual Studio 2012, just install the PowerCommands for Visual Studio 2010 again and it will offer to add it to Visual Studio 2012. You will still need to make the shortcut assignment in 2012.

[![Visual Studio Keyboard Options (Edit.UndoClose)](/wp-content/uploads/2012/02/UndoCloseSettings-300x174.png "Getting Ctrl+Shift+T to re-open previously closed tabs/windows.")](/wp-content/uploads/2012/02/UndoCloseSettings.png)

Just like with your favorite web browser, this will cycle through previously closed tabs, resurrecting them from their graves and restoring your cursor location to right where you left off when you closed them. Amazingly enough, this also works when Visual Studio closes your active windows when a project reloads.

---

## Comments

* **Sergey Vlasov**, _2012-02-15 14:23:54 +0000_

    > PowerCommands extension for Visual Studio 2010 lets you bring back the last closed window: http://visualstudiogallery.msdn.microsoft.com/e5f41ad9-4edc-4912-bca3-91147db95b99/
    >
    > "To reopen the most recently closed document, point to the Edit menu, then click Undo Close. Alternately, you can use the Ctrl+Shift+Z shortcut."

    * **Adam Patridge**, _2012-02-15 16:21:59 +0000_

        > Sergey, thanks for tossing that out. That's a good extension to know. I haven't given that one a chance, though I do remember hearing about it when someone asked about recursively collapsing projects in the Solution Explorer. There's a lot to love in that extension (e.g., "Remove and Sort Usings on save"). I appreciate that they make it easy to disable during a presentation, too, if I want to avoid confusing people who may not have it installed (at the risk of looking silly when I try to use its [now disabled] features).

* **Kurt**, _2012-08-04 03:55:06 +0000_

    > Thanks for this. There are a few silly things in VS 2010. Not showing line numbers by default is another...

    * **Adam Patridge**, _2012-08-24 17:33:11 +0000_

        > I definitely use line numbers fairly regularly, but that one must not bother me enough to turn it on. If I need a current line number, by default, the cursor's line number is shown on the IDE status bar (bottom right: "Ln ##"; still there in VS2012). Going the other way, any time I need to find a line by number, I just mash the Ctrl+G shortcut for "Go To...".

* **Luke Sampson**, _2012-08-24 00:37:13 +0000_

    > Sweet! I tried setting Ctrl-W as a global command but it didn't work—thanks for the 'Text Editor' tip!

    * **Adam Patridge**, _2012-08-24 17:26:09 +0000_

        > Glad it helped. I tried that way first, too. There may be window types other than "Text Editor" that wouldn't allow the shortcut to work this way (maybe some designers or explorer-style windows), but I haven't found them yet.

* **Hio**, _2013-10-26 13:09:57 +0000_

    > Not working in VS 2013

    * **Adam Patridge**, _2013-11-08 16:53:50 +0000_

        > Valid point. Even [Productivity Power Tools](http://visualstudiogallery.msdn.microsoft.com/d0d33361-18e2-46c0-8ff2-4adea1e34fef), which had this feature and supported 2012, doesn't appear to offer support for 2013. Here's hoping someone on [Stack Overflow can let us know of an undo-close solution for VS2013](http://stackoverflow.com/q/19864290/48700).

        * **Adam Patridge**, _2013-11-18 04:25:01 +0000_

            > They just released a Productivity Power Tools for VS 2013 a couple days ago. I added an update at the beginning of the post with a link.

* **Mike**, _2015-07-02 19:53:31 +0000_

    > Just dropping a quick thanks. That's been bugging me for a long time now.

* **vijay**, _2015-10-02 15:47:26 +0000_

    > thank you very much!
    > I wish some day all these companies agree to put people before product and use same short cuts for common tasks.

* **Anton Vlasiuk**, _2019-02-26 18:00:33 +0000_

    > Thank you so much! Wonderful tutorial.

