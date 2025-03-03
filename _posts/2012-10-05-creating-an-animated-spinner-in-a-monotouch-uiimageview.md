---
layout: post
title: "Creating an animated spinner in a Xamarin.iOS (MonoTouch) UIImageView"
author: "Adam Patridge"
date: Fri, 05 Oct 2012 18:59:49 +0000
tags: xamarin uiimageview xamarin-ios monotouch
category: dev
excerpt_separator: <!--more-->
---

## Background

I'm well into my first week of building the Sierra Trading Post first iOS app using [Xamarin.iOS](http://xamar.in/r/SierraTradingPost/xamarin.com/monotouch) and it has been a fun ride so far. One of the first things needed was a system for showing a loading image while asynchronously retrieving the final image with a web request.

<!--more-->

## Attempt 1

Xamarin has a recipe for using a `UIImageView`'s `AnimationImages` to make a spinner.

    UIImageView someImageView = new UIImageView();
    someImageView.AnimationImages = new UIImage[] {
        UIImage.FromBundle("Spinning Circle_1.png"),
        UIImage.FromBundle("Spinning Circle_2.png"),
        UIImage.FromBundle("Spinning Circle_3.png"),
        UIImage.FromBundle("Spinning Circle_4.png"),
    };
    someImageView.AnimationRepeatCount = 0; // Repeat forever.
    someImageView.AnimationDuration = 1.0; // Every 1s.
    someImageView.StartAnimating();

It may be possible to make this work, but it wasn't quite what I needed. This seems to be more of an image rotation than an animation. As a result, it creates a jerky animation between the various images equally distributed over the `AnimationDuration` you set.

After this, attempts to find some ideas for a better solution lead me to about a hundred lines of [code that proved a difficult to consume](http://sabonrai.wordpress.com/2009/09/03/monotouch-sample-core-graphics-and-uiimageview/), involving a `CGBitmapContext` and `CGAffineTransform.MakRotation`. (To be fair, this code isn't doing something as simple as what I want to do.) Hoping to avoid that, I simply added four more rotation positions into the list. It would probably take many more to make it appear smooth. Any more than that and I really didn't want to bloat my project with minute rotations of the same PNG. Back to Google I went.

## Solution

After enough poking around some [slightly](http://stackoverflow.com/questions/10054693/how-to-create-smooth-image-animation) [related](http://docs.xamarin.com/ios/recipes/Animation/CoreAnimation/Create_an_Animation_Group) Google results, I began to understand enough of `CABasicAnimation` to see how it could work for the job. You create the desired animation and add an instance of `UIImageView.Layer`.

        // Image to be rotated (in this case, found in the project as "/Assets/Images/loading_icon.png").
        UIImageView someImageView = new UIImageView(UIImage.FromBundle("Assets/Images/loading_icon"));
        CABasicAnimation rotationAnimation = CABasicAnimation.FromKeyPath("transform.rotation");
        rotationAnimation.To = NSNumber.FromDouble(Math.PI * 2); // full rotation (in radians)
        rotationAnimation.RepeatCount = int.MaxValue; // repeat forever
        rotationAnimation.Duration = 1;
        // Give the added animation a key for referencing it later (to remove, in this case).
        someImageView.Layer.AddAnimation(rotationAnimation, "rotationAnimation");

The main part of this is the simple rotation `CABasicAnimation` that is applied to a `Layer` of a `UIImageView`. In this case, it is set to do a full rotation (accepted in radians) every one second through a very large number of repetitions. The repetitions is actually one oddity in the switch to this new method. When you set `UIImageView.AnimationRepeatCount`, you can set it to zero to make it loop forever. Oddly, a `CABasicAnimation.RepeatCount` set to zero is the same as one, and it loops a single time before stopping.

## Code (Example doing for a bunch of UITableView cells)

    static NSString key = new NSString("somecellkey");
    public override UITableViewCell GetCell(UITableView tableView, NSIndexPath indexPath) {
        UITableViewCell cell = tableView.DequeueReusableCell(key);
        if (cell == null) {
            cell = new UITableViewCell(UITableViewCellStyle.Default, key);
        }

        // Image to be rotated (in this case, found in the project as "/Assets/Images/loading_icon.png").
        cell.ImageView.Image = UIImage.FromBundle("Assets/Images/loading_icon");
        CABasicAnimation rotationAnimation = CABasicAnimation.FromKeyPath("transform.rotation");
        rotationAnimation.To = NSNumber.FromDouble(Math.PI * 2); // full rotation (in radians)
        rotationAnimation.RepeatCount = int.MaxValue; // repeat forever
        rotationAnimation.Duration = 1;
        // Give the added animation a key for referencing it later (to remove, in this case).
        cell.ImageView.Layer.AddAnimation(rotationAnimation, "rotationAnimation");

        // Do your lazy-loading of the image (blog post coming soon...maybe).
        ...

        // For a good time, you can keep rotating your final, lazy-loaded image by not calling this line.
        cell.ImageView.Layer.RemoveAnimation("rotationAnimation");

        // Do the rest of your visual stuff to the cell.
        ...

        return cell;
    }

## More Code

If you want a quick demo application of the differences, check out the [GitHub repo](https://github.com/patridge/SpinnningUIImageView) I put together [and finally got around to sharing]. It is a simple demo of two `UIImageView`s that implement the two methods here. Clicking anywhere will toggle between the two.

Here's a quick video snippet of the demo code running.

<iframe width="420" height="315" src="http://www.youtube.com/embed/vXp3akYh1Hk" frameborder="0" allowfullscreen></iframe>
