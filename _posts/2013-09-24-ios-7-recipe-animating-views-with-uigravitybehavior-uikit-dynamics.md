---
layout: post
title: "Xamarin.iOS C# Recipe: Animating Views with iOS 7 UIGravityBehavior (UIKit Dynamics)"
author: "Adam Patridge"
date: Wed, 25 Sep 2013 05:23:23 +0000
tags: Xamarin iOS-7 iOS Xamarin.iOS monotouch
category: dev
excerpt_separator: <!--more-->
---

<div style="float: right; padding-left: 15px;"><img style="width: 200px;" alt="UIGravityBehavior, oddly addictive" src="/wp-content/uploads/2013/09/ChangingUIGravityBehavior.gif"></div>

Now that iOS 7 has landed, and [Xamarin gave us same-day C# support](http://blog.xamarin.com/ios-7-and-xamarin-ready-when-you-are/), it's time to start poking at the new bits. One such piece is UIKit Dynamics. With UIKit Dynamics, you can greatly simplify all sorts of view animations. While this simple recipe will only address `UIGravityBehavior`, iOS 7 adds a bunch of other predefined behaviors and allows the creation of custom ones as well.

<!--more-->

Source code is available on [GitHub](https://github.com/patridge/UIGravityBehaviorRecipe).

## Basic Gravity Animation

To have a view simply start falling off the screen, you just toss the desired view at a new `UIGravityBehavior` and add that behavior to a `UIDynamicAnimator`.

    UIDynamicAnimator animator;
    public override void ViewDidLoad() {
        base.ViewDidLoad();
        View.BackgroundColor = UIColor.White;
        animator = new UIDynamicAnimator(View);

        var item = new UIView(new RectangleF(new PointF(50f, 0f), new SizeF(50f, 50f))) {
            BackgroundColor = UIColor.Blue,
        };
        View.Add(item);
        UIGravityBehavior gravity = new UIGravityBehavior(item);
        animator.AddBehavior(gravity);
    }

![Basic UIGravityBehavior](/wp-content/uploads/2013/09/BasicUIGravityBehavior.gif)

You can continue to put items under the effect of gravity by adding them to the behavior.

    gravity.AddItem(someOtherView);

That's really it for basic gravity, but craziness is only a step beyond that. You can modify the `Angle`, `GravityDirection`, and `Magnitude` of your gravity as well.

### Potential Memory/Performance Trap

It's worth noting that those items that go flying off the screen under the effects of gravity will continue to keep falling, even though they are no longer rendered on the screen. If you need to worry about resources, you could be computing locations for things long since forgotten.

## Customizing Gravity

The three `UIGravityBehavior` properties, `Angle`, `GravityDirection`, and `Magnitude`, can all be changed to your liking and they are interconnected, changing one can affect the others.

### Magnitude

The default gravity acceleration is 1000 pixels/sec^2. You could very easily bump the gravity to Jupiter levels (roughly 2.5 times that of Earth).

    UIGravityBehavior gravity = new UIGravityBehavior(item) {
        Magnitude = 25000,
    };

### Direction (`GravityDirection` and `Angle`)

You can change the direction of gravity with two different interconnected variables, `Angle` and `GravityDirection`; change one and you affect the other. The default gravity angle pulls straight down, as measured in [radians](http://en.wikipedia.org/wiki/Radians) from the righthand side: `PI / 2` radians, approximately 1.57. This is also expressed by the gravity direction vector, the default being `(0, 1)`. Gravity's direction can be adjusted to your whim, and reversing it is quite simple using a vector pointing the opposite direction.

    UIGravityBehavior gravity = new UIGravityBehavior(item) {
        GravityDirection = new CGVector(0, -1),
    };

Alternatively, with more math involved, you can set the radian angle. 

    UIGravityBehavior gravity = new UIGravityBehavior(item) {
        Angle = -(float)Math.PI / 2,
    };

### Action (Interacting with the Animation)

While not strictly a property for modifying gravity, `UIGravityBehavior`, like all the other behaviors in UIKit Dynamics, allows you to execute some code for each animation frame using its `Action`. You could use it for debugging a wonky behavior or for cleaning up views that you may have let fly off the screen.

    UIGravityBehavior gravity = new UIGravityBehavior(item) {
        Action = () => {
            Console.WriteLine(item.Frame.Location);
        }
    };

## Bending Gravity to your Will (Advanced Recipe)

Having control over gravity can quickly go to your head. In the off chance you need to change gravity's effect on your UI elements on demand, check out the secondary controller in the [demo code](https://github.com/patridge/UIGravityBehaviorRecipe/blob/master/UIGravityBehaviorRecipe/ChangingGravityViewController.cs). In this proof-of-concept, you tap to place an object under gravity's control, simple enough with a `UITapGestureRecognizer`.

    View.AddGestureRecognizer(new UITapGestureRecognizer((gesture) => {
        PointF tapLocation = gesture.LocationInView(View);
        var item = new UIView(new RectangleF(PointF.Empty, sizeInitial)) {
            BackgroundColor = ColorHelpers.GetRandomColor(),
            Center = tapLocation,
        };
        items.Enqueue(item);
        View.Add(item);
        gravity.AddItem(item);

        // ...
    }));

Any time you want, though, you can swipe to alter its direction using a series of `UISwipeGestureRecognizer`s (each one can only recognize a single swipe direction).

    View.AddGestureRecognizer(new UISwipeGestureRecognizer((gesture) => {
        gravity.GravityDirection = new CGVector(1, 0);
    }) { Direction = UISwipeGestureRecognizerDirection.Right, });
    View.AddGestureRecognizer(new UISwipeGestureRecognizer((gesture) => {
        gravity.GravityDirection = new CGVector(-1, 0);
    }) { Direction = UISwipeGestureRecognizerDirection.Left, });
    View.AddGestureRecognizer(new UISwipeGestureRecognizer((gesture) => {
        gravity.GravityDirection = new CGVector(0, -1);
    }) { Direction = UISwipeGestureRecognizerDirection.Up, });
    View.AddGestureRecognizer(new UISwipeGestureRecognizer((gesture) => {
        gravity.GravityDirection = new CGVector(0, 1);
    }) { Direction = UISwipeGestureRecognizerDirection.Down, });

I'm not sure I would call it a "game", but I will admit to spending far more time poking at the screen than was necessary.

![Controlling UIGravityBehavior by swipe](/wp-content/uploads/2013/09/ChangingUIGravityBehavior.gif)

## Keep Learning

Xamarin has put together a fantastic [C# iOS 7 introduction](http://docs.xamarin.com/guides/ios/platform_features/introduction_to_ios_7) and a great [overview on iOS 7 UI changes](http://docs.xamarin.com/guides/ios/platform_features/ios7_ui).  Expect more great iOS 7 C# tutorials to come out of [Xamarin's docs team](http://docs.xamarin.com/) as well as others participating in the [Xamarin Recipe Cook-off](http://blog.xamarin.com/xamarin-recipe-cook-off/). (Disclaimer: I am probably getting a free Xamarin t-shirt for this post.)

UIKit Dynamics are far more than just this one pre-baked gravity behavior. I highly recommend checking out the 2013 WWDC videos: [Getting Started with UIKit Dynamics](https://developer.apple.com/wwdc/videos/?id=206) and [Advanced Techniques with UIKit Dynamics](https://developer.apple.com/wwdc/videos/?id=221).
