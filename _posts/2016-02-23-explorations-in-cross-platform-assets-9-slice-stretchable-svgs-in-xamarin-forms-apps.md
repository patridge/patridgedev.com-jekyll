---
layout: post
title: "Explorations in Cross-Platform Assets: 9-Slice Stretchable SVGs in Xamarin.Forms Apps"
date: Tue, 23 Feb 2016 14:10:21 +0000
tags: Xamarin.Forms
# category: dev
excerpt_separator: <!--more-->
---

There comes a time in cross-platform development where you inevitably long for a simpler mechanism for dealing with the deluge of image assets. You end up with three or more sizes of every single image with different names for every platform. At Twin, SVGs have risen to that challenge for us, and we use it everywhere we can with a custom variant of [Paul Patarinski's SvgImage control](https://github.com/paulpatarinski/Xamarin.Forms.Plugins/tree/master/SVG). After some minimal setup work to start using them, they render crisply at any size and resolution we throw at them.

<!--more-->

One place that SVG use has had shortcomings for us is when we needed backgrounds with varying sizes. We need a unique SVGs for each size we intend to use because you haven't been able to selectively stretch an SVG like you could with [9-patch images](http://developer.android.com/tools/help/draw9patch.html) on Android or [StretchableImage](http://developer.xamarin.com/api/member/UIKit.UIImage.StretchableImage/)/[CreateResizableImage](https://developer.xamarin.com/api/member/MonoTouch.UIKit.UIImage.CreateResizableImage/p/MonoTouch.UIKit.UIEdgeInsets/) on iOS. In SVG terms, this appears to be called 9-slice support, and it is something we want to both use in our apps as well as make available to you. I recently took an experimental journey toward bringing 9-slice support to SVGs drawn in cross-platform Xamarin.Forms apps.

Read the full [9-Slice SVG article over at Twin Technologies blog](https://web.archive.org/web/20160814135520/http://blog.twintechs.com/explorations-in-cross-platform-assets-xamarin-forms) [archived copy; images lost to time, sadly].
