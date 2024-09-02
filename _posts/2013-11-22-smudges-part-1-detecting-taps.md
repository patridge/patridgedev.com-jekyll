---
layout: post
title: "Abusing UIKit for Gaming in Xamarin.iOS, Part 1: Detecting Taps and Placing Views with UITapGestureRecognizer"
date: Sat, 23 Nov 2013 01:27:08 +0000
tags: Xamarin Smudges UIKit UITapGestureRecognizer Xamarin.iOS
# category: dev
excerpt_separator: <!--more-->
---

_This is the first in a series of Abusing UIKit blog posts giving some background on the development that want into producing [Smudges](/smudges/), a simple game written entirely in Xamarin.iOS where fun shapes in various colors show up on the screen wherever a tap is detected. It was original created to give my two-year-old something fun to play while going tap-crazy on the screen. The game evolved from those "play-testing" sessions. If you have your own little ones and want something fun to distract them, [Smudges is availabe on the App Store](https://itunes.apple.com/us/app/smudges/id739618884?mt=8&uo=4&ct=blog). At this point, I plan to continue adding features to it as I can. Let me know what you think about Smudges, or these blog posts, in the comments below or [find @patridgedev on Twitter](https://twitter.com/patridgedev/)._

<div style="float: right; padding-left: 10px;"><img style="width: 200px;" src="/wp-content/uploads/2013/11/PlacingViewsDemo.gif" alt="Demo animation of placing views with a UITapGestureRecognizer." /></div>

## Where Did They Touch?

Smudges has a simple game mechanic: tap the screen, new shape appears. The first step is figuring out when and where a tap occurred. The simple approach is to put a `UIButton` where you need to detect a touch, attaching a handler to its `TouchUpInside` event.

<!--more-->

    UIButton someButton = new UIButton(someFrameRectangle);
    AddSubview(someButton);
    someButton.TouchUpInside += (sender, e) => {
        Debug.WriteLine("Touched somewhere on this button.");
    };

Of course, while incredibly simple, this is only practical for interface elements; it's not so great for doing something at the exact location of a tap. To know the X-Y location where the user's finger is touching the screen, you need to wander into `UITapGestureRecognizer` territory (or beyond).

    UITapGestureRecognizer tapRecognizer = new UITapGestureRecognizer() {
        NumberOfTapsRequired = 1, // default is 1, adjust as needed
    };
    tapRecognizer.AddTarget((sender) => {
        PointF location = tapRecognizer.LocationOfTouch(0, MainView);
        Debug.WriteLine("Touched: {0}", location);
    });
    someContainerView.AddGestureRecognizer(tapRecognizer);

Simple enough as well, and a great choice for double-taps...and, I suppose, three-finger triple-taps, if you want to get crazy.

## Placing the View

Once you have the tap location, doing something with it is fairly trivial.

    tapRecognizer.AddTarget((sender) => {
        PointF location = tapRecognizer.LocationOfTouch(0, MainView);
        UIView newView = new UIView(new RectangleF(location, new SizeF(50f, 50f))) {
            BackgroundColor = UIColor.Red,
        };
        someContainerView.AddView(newView);
    });

This will happily place a 50x50 rectangle on the screen with its top-left corner sitting right where the tap was detected. Centering on that location was my goal, which is easy enough by setting the new view's `Center` instead of a full frame.

    tapRecognizer.AddTarget((sender) => {
        PointF location = tapRecognizer.LocationOfTouch(0, MainView);
        UIView newView = new UIView() {
            Size = new SizeF(50f, 50f),
            Center = location,
            BackgroundColor = UIColor.Red,
        };
        someContainerView.AddView(newView);
    });

## Make it Pretty

One thing I tend to do in demos of any kind is introduce some randomness to the mix. It keeps things much more interesting to everyone, especially me while I am running it for the thousandth time to get everything just right. In this case, let's pick a random color for each one.

    static Random rand = new Random();
    public static UIColor GetRandomColor() {
        int red = rand.Next(255);
        int green = rand.Next(255);
        int blue = rand.Next(255);
        UIColor color = UIColor.FromRGBA(
            (red / 255.0f),
            (green / 255.0f),
            (blue / 255.0f),
            1f);
        return color;
    }

If you need random numbers that hold up over time, you will want to investigate better random number generators, but `System.Random` serves its purpose well for these uses. Throwing all that into the mix, we now get a new "fun" gesture recognizer target that places a rectangle of a random color centered right at the touch location.

    UITapGestureRecognizer tapRecognizer = new UITapGestureRecognizer() {
        NumberOfTapsRequired = 1
    };
    tapRecognizer.AddTarget((sender) => {
        PointF location = tapRecognizer.LocationOfTouch(0, MainView);
        UIView newView = new UIView() {
            Size = new SizeF(50f, 50f),
            Center = location,
            BackgroundColor = GetRandomColor(),
        };
        someContainerView.AddView(newView);
    });
    someContainerView.AddGestureRecognizer(tapRecognizer);

## Caution: Memory Leak Ahead

When you go adding views all the time, with enough time, you are bound to run up against memory pressures. The simplest solution is to keep track of added views so you can remove them later. In this demo, I simply schedule a task to do so after some time elapses. In a later demo, I will have a simple demo using a `Queue` that does just that. Another option is a pool of views, something many game engines offer directly. There's lots of room for turning this potential limitation into something fun.

## Beyond `UITapGestureRecognizer`

This version allows the detection of a single finger tapping. If you use two fingers to tap at the same time, nothing will happen. In a later post, I will address recognizing simultaneous touches.

## Demo Source Code

For details on the view queueing system or any of the rest of this post's code, just pop over to the [GitHub repo for the Abusing UIKit for Gaming in Xamarin.IOS series](https://github.com/patridge/UIKitAbuse). I'll be adding new projects to that solution as I add posts to this series.

## One last shameless plug...

Find [Smudges on the App Store](https://itunes.apple.com/us/app/smudges/id739618884?mt=8&uo=4&ct=blog), available for iPhone, iPad, or iPod Touch. It's especially fun on iPad where the hardware allows for ten simultaneous touch points.
