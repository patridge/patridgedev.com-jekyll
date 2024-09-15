---
layout: post
title: "Closing and Re-opening tabs in Visual Studio with Ctrl+W"
author: "Adam Patridge"
date: Tue, 14 Feb 2012 16:13:17 +0000
tags: keyboard-shortcut Visual-Studio
# category: dev
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
