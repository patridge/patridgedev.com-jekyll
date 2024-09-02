---
layout: post
title: "Capturing Your iOS App in Animated GIF Glory using LICEcap"
date: Fri, 27 Sep 2013 13:30:55 +0000
tags: Xamarin iOS Xamarin.iOS
# category: dev
excerpt_separator: <!--more-->
---

<div style="float: right; padding-left: 10px;"><img style="width: 200px;" src="/wp-content/uploads/2013/09/LICEcapHighlight.gif" alt="LICEcap Capture at 30fps" /></div>

Showing the coolness of your iOS app in a web format can be very difficult, depending on what about your app makes it shine. If your app thrives on animation, especially the new UIKit Dynamics fun, you will need more than one frame to portray what your app does: enter the animated GIF, mother of all awesomeness.

<!--more-->

Here is the method I used to make the images for my [Xamarin UIGravityBehavior recipe](http://pdev.co/19CZ0DJ). That said, if you know a better way to do this, toss a comment out there; I'd love to hear about it.

*While I use these methods for my Xamarin.iOS creations, they apply equally to native apps and most anything running on a Mac.*

## The Tool: LICEcap

While the name sounds slightly...off, it definitely gets the job done. [LICEcap](http://www.cockos.com/licecap/), from Cockos Incorporated, is a quick disk image install. In fact, here's LICEcap's capture of me installing LICEcap.

![Installing LICEcap](/wp-content/uploads/2013/09/InstallingLICEcap.gif)

One thing to notice, these GIFs are not a great way to capture complex color palettes like the gradients in the install image folder. There may be a tool that is better at that, but you will probably compromise file size for fidelity. That install GIF was 81kb.

### Capturing the iOS Simulator

With LICEcap, you capture an arbitrary portion of the screen, so you can capture just a portion of your app all the way to the full simulator with your Twitter feed rolling in the background around it. LICEcap loads as a transparent window, looking eerily as if something failed to render. Instead, what the program frames is what will be captured. You pull the various window edges and corners as needed to frame your target. It can be maddening to adjust it just right if you are shooting for specific pixels. You can also set exact pixel measurements with the two bottom inputs.

![Using LICEcap to capture an animated GIF](/wp-content/uploads/2013/09/UsingLICEcap.png)

After you set your capture area, you can set a maximum frames per second (FPS) as well. For representative purposes, it seemed I needed fewer FPS than I thought. The resulting file size variation from FPS settings will depend heavily on your recording length and the complexity of the graphics. For my UIGravityBehavior recipe, practically ideal for GIF compression, it wasn't a big difference from 5fps to 30fps. For some animations, file size can get substantial fairly quickly.

<div style="float: right; padding-left: 10px;"><img style="width: 125px;" src="/wp-content/uploads/2013/09/LICEcapCapture30fps.gif" alt="LICEcap Capture at 30fps" /></div>
<div style="float: right; padding-left: 10px;"><img style="width: 125px;" src="/wp-content/uploads/2013/09/LICEcapCapture05fps.gif" alt="LICEcap Capture at 5fps" /></div>
<table style="width: 50%;">
  <tr><th>FPS</th><th>File Size</th></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture05fps.gif">5fps</a></td><td>24kb</td></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture10fps.gif">10fps</a></td><td>43kb</td></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture15fps.gif">15fps</a></td><td>46kb</td></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture20fps.gif">20fps</a></td><td>66kb</td></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture25fps.gif">25fps</a></td><td>67kb</td></tr>
  <tr><td><a href="/wp-content/uploads/2013/09/LICEcapCapture30fps.gif">30fps</a></td><td>68kb</td></tr>
</table>

Unless you want a huge recording, you probably want to scale the Simulator to 50% with <kbd>Command</kbd>+<kbd>3</kbd>. For a full-app capture, you will end up getting the bottom rounded corners of the window unless you crop a few pixels. For most apps, trimming those pixels doesn't seem to be a loss. At half scale, I find setting LICEcap to 318x564 then lining up with a top corner works quite well. Doing so highlights one frustration with the program. When capturing skinny views with exact dimensions, such as the iOS Simulator in portrait, you need to enter your height first or the field disappears before you can give it a value.

Once you are all set, hit the record button. Give your file a name and a location. You will also have a few options here that may be of interest, such as showing a circle for mouse button presses or loop repeat count. When you are all set hit "Save" and you will get a 3-2-1 countdown in the title bar before recording begins. After everything is complete, simply hit "Stop" to save the result. (You can also pause and resume during a recording.)

### Trimming the result

If you find you have a bunch of extra frames in your animation, open your GIF in Preview. From there you can selecting the offending frames and hit delete. But, once you edit in Preview, any infinite looping you had will be gone. I found [Gifsicle](http://www.lcdf.org/gifsicle/), a command line utility, fixed this for me quite nicely.

    gifsicle -bl ~/path/to/your/file.gif

## Other Capture Methods

### Still Images

If you really just need a static screenshot, don't forget the baked-in offerings. The easiest shots can be done in the iOS Simulator, a simple <kbd>Command</kbd>+<kbd>S</kbd> (File: Save Screen Shot) will save a PNG result of your app to the desktop.

Need to capture the Simulator among other windows? You capture the full screen with <kbd>Command</kbd>+<kbd>Shift</kbd>+<kbd>3</kbd> and can then crop to your desired size. To capture a partial screenshot, hit <kbd>Command</kbd>+<kbd>Shift</kbd>+<kbd>4</kbd> and draw what to capture with the crosshairs. These images are also saved to your desktop, time-stamped, but they are formatted "Screen Shot {yyyy-MM-dd} at {h.mm.ss tt}".

### Capturing a Video

If you need a video to do your app justice, by all means, let me introduce you to this neat tool included on your Mac for free: QuickTime Player. Load it up and select "File", then "New Screen Recording". Here's where I would embed a video of me doing that, but you can't start a new screen recording while doing a screen recording.

![Using QuickTime Player for Screen Recordings](/wp-content/uploads/2013/09/QuickTimeScreenRecording.png)

## Other Options?

Utilities such as [Jing](http://www.techsmith.com/jing.html) capture in a Flash-based format, but I have mixed feelings about requiring Flash, no matter how common it may be. While the animation captures I tried with it were smooth and the color gradients captured fine, the file sizes got substantial quickly.

If there is some awesome, affordable or free tool out there that you use to make capturing animations insanely easy? I'd love to hear about it.
