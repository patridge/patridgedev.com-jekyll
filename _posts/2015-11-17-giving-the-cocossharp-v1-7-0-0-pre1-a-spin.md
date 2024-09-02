---
layout: post
title: "Giving CocosSharp v1.7.0.0-pre1 a Spin"
date: Tue, 17 Nov 2015 17:18:36 +0000
tags: Xamarin.Forms CocosSharp
# category: dev
excerpt_separator: <!--more-->
---

<div style="float: right; padding-left: 10px;"><a href="/wp-content/uploads/2015/11/ios-cocossharp-game-listview-hybrid.png"><img src="/wp-content/uploads/2015/11/ios-cocossharp-game-listview-hybrid-169x300.png" alt="iOS screenshot of a hybrid screen showing a Xamarin.Forms ListView and a CocosSharp game splitting the screen." width="169" height="300" class="alignnone size-medium wp-image-489" /></a></div>

If you didn't see the announcement recently, the awesome people working on CocosSharp put out a new preview of CocosSharp v1.7 (v1.7.0.0-pre1). With it, comes a very cool new way of working with your game content, the ability to mix it with normal native Xamarin.iOS, Xamarin.Android, or Xamarin.Forms content. I wanted to poke at the last of those, because I've been buried in Xamarin.Forms code non-stop for quite a few months now.

## TL;DR:

Getting the CocosSharp project templates working the the v1.7.0.0-pre1 packages takes a few extra steps: restoring the pre-release CocosSharp.Forms NuGet package on your shared *and* platform projects. Once you get things in place, though, you can do some amazing things by combining Xamarin.Forms content with your CocosSharp game content.

If you want to just jump right into playing with the final code, the resulting solution is available in a [GitHub repo](https://github.com/patridge/CocosSharpLovesXamarinForms/). (I may continue to work on this repo, so if it ever looks too different, here is the [exact commit to clone](https://github.com/patridge/CocosSharpLovesXamarinForms/tree/d7bb965ee33bc5b6bfcc546ca248a12f400c4240) to get you to the end of this post.)

<div style="clear:both;" />

<!--more-->

### CocosSharp Project Templates Add-in

While we can spin up a new Xamarin.Forms project and add the CocosSharp bits, this also gives us a chance to play with a nice Xamarin Studio add-in. Open up the Add-in Manager from the Xamarin Studio menu and search for "cocos" (technically "CocosSharp project templates") and give the Install… button a click. Let's restart Xamarin Studio to let it pick up the new templates.

<a href="/wp-content/uploads/2015/11/xamarin-studio-add-in-manager.png"><img src="/wp-content/uploads/2015/11/xamarin-studio-add-in-manager-300x203.png" alt="Adding the CocosSharp project templates add-in." width="300" height="203" class="alignnone size-medium wp-image-492" /></a>

When Xamarin Studio comes back up, create a New Solution. In the New Project window, under Cross-platform > App, you should now see two new CocosSharp entries: CocosSharp.Forms Game and CocosSharp.Forms.Game Showcase. The former is just an empty project pre-wired to try to get plumbed up for CocosSharp (more on that later), and the later is a similar project but with a little sample game content. We will pick the Showcase one so we have something on the screen immediately. Select that template and hit Next. Give your sample app some basic information like App Name. (I left everything else with the defaults, like PCL for shared code and Git integration, but feel free to customize.)

### Package Restore Failure

Once you let Xamarin Studio spin up all the parts of your new solution, you'll be presented with one of the hiccups with playing around with pre-release libraries.

<a href="/wp-content/uploads/2015/11/cocossharp-project-template-package-restore-fail-message.png"><img src="/wp-content/uploads/2015/11/cocossharp-project-template-package-restore-fail-message-300x16.png" alt="Error restoring pre-release NuGet packages on the CocosSharp project templates." width="300" height="16" class="alignnone size-medium wp-image-488" /></a>

By default, the NuGet package restore on our new project will fail because the CocosSharp packages it references are still in pre-release mode. If you open up the Package Console pad, you'll see a message like this. (Note: when v1.7.0.0-pre1 is updated to go live, this won't be an issue anymore.)

> Adding CocosSharp.Forms...
> Attempting to resolve dependency 'Xamarin.Forms'.
> Attempting to resolve dependency 'CocosSharp'.
> Unable to resolve dependency 'CocosSharp'.

If you don't even notice this error and try to compile the project anyway, you'll get various build errors as it tries and fails to resolve various CocosSharp namespaces.

> CocosLovesXamarinForms.cs(7,7): Error CS0246: The type or namespace name `CocosSharp' could not be found. Are you missing an assembly reference? (CS0246) (CocosLovesXamarinForms)

We'll have to add the NuGet packages ourselves. It's not that the project templates are broken, really, they just need some help to work with the system as it is.

### Adding Pre-Release Packages

Double-click on the Packages folder of your shared PCL project to open the Add Package dialog. Since we know we want a pre-release package, check the bottom-left box to "Show pre-release packages." Then, search for "CocosSharp.Forms" and add the CocosSharp for Xamarin.Forms package to that project.

<a href="/wp-content/uploads/2015/11/xamarin-studio-add-package-pre-release-cocossharp-forms.png"><img src="/wp-content/uploads/2015/11/xamarin-studio-add-package-pre-release-cocossharp-forms-300x198.png" alt="Adding the pre-release CocosSharp NuGet  package." width="300" height="198" class="alignnone size-medium wp-image-493" /></a>

Now, your PCL project will build, but you will still see errors on your platform projects because they are missing the Xamarin.Forms packages. This is slightly confusing because we don't need pre-release Xamarin.Forms packages. So, why wouldn't those be restored? Well, they weren't added to these projects directly. They were supposed to be added by way of the dependencies on the CocosSharp.Forms package. Since that didn't restore, we didn't get its dependencies either, apparently.

### If You Only See White

As a warning, if you run off and just add Xamarin.Forms to the platform project(s), you will end up with projects that build and run "without errors." Unfortunately, you won't have anything but white on the screen either.

<a href="/wp-content/uploads/2015/11/ios-cocossharp-template-project-without-renderers.png"><img src="/wp-content/uploads/2015/11/ios-cocossharp-template-project-without-renderers-169x300.png" alt="Running the CocosSharp template project without restoring the platform CocosSharp.Forms packages." width="169" height="300" class="alignnone size-medium wp-image-491" /></a>

Because the CocosSharp.Forms package contains the custom platform renderers for the `CocosSharpView` used in the PCL code, without them you don't see anything on iOS or Android.

### CocosSharp.Forms All the Things

To get your game rendering on the platform projects, we also need to add the CocosSharp.Forms NuGet package to the iOS and Android platform projects the same way we did for the shared PCL project. (Make sure you check the "Show pre-release packages" checkbox each time to find it.)

With those packages in place, you can see your project template game in all its glory.

<a href="/wp-content/uploads/2015/11/android-cocossharp-template-project-running.png"><img src="/wp-content/uploads/2015/11/android-cocossharp-template-project-running-180x300.png" alt="Android screenshot of the CocosSharp project template game." width="180" height="300" class="alignnone size-medium wp-image-487" /></a>

<a href="/wp-content/uploads/2015/11/ios-cocossharp-template-project-running.png"><img src="/wp-content/uploads/2015/11/ios-cocossharp-template-project-running-169x300.png" alt="iOS screenshot of the CocosSharp project template game." width="169" height="300" class="alignnone size-medium wp-image-490" /></a>

### Mixing UI and Game Worlds

Now that your CocosSharp game isn't bound to the world of running only in full-screen, you can do some fun and crazy things. For example, if you want to mix a game with a Xamarin.Forms ListView, we can now do that.* (* Some restrictions apply.)

Let's open up GamePage.cs and start making some adjustments (<kbd>Cmd</kbd>+<kbd>.</kbd>, "GamePage.cs", <kbd>Enter</kbd>). Currently, the constructor is just putting the game in place.

    CocosSharpView gameView;
    
    public GamePage ()
    {
    	gameView = new CocosSharpView () {
    		HorizontalOptions = LayoutOptions.FillAndExpand,
    		VerticalOptions = LayoutOptions.FillAndExpand,
    		// Set the game world dimensions
    		DesignResolution = new Size (1024, 768),
    		// Set the method to call once the view has been initialised
    		ViewCreated = LoadGame
    	};
    
    	Content = gameView;
    }

Because of the way things work with `CocosSharpView`, it has to know ahead of time what size to be; you won't be able to just request something like `LayoutOptions.FillAndExpand`. When I need to pre-determine dimensions and still handle multiple layouts, I tend to grab for an `AbsoluteLayout` and then do the necessary layout in the `LayoutChildren` overrride.

    CocosSharpView gameView;
    ListView listView;
    AbsoluteLayout mainAbsoluteLayout;
    
    public GamePage ()
    {
    	listView = new ListView () {
    		ItemsSource = "These are some words I want to display in a list".Split (new[] { ' ' }),
    	};
    	gameView = new CocosSharpView () {
    		// Set the game world dimensions
    		DesignResolution = new Size (1024, 768),
    		// Set the method to call once the view has been initialised
    		ViewCreated = LoadGame
    	};
    	mainAbsoluteLayout = new AbsoluteLayout () {
    		Children = {
    			listView,
    			gameView,
    		},
    	};
    
    	Content = mainAbsoluteLayout;
    }
    
    protected override void LayoutChildren (double x, double y, double width, double height)
    {
    	base.LayoutChildren(x, y, width, height);
    
    	AbsoluteLayout.SetLayoutBounds (listView, new Rectangle (0, 0, width, height / 2));
    	AbsoluteLayout.SetLayoutBounds (gameView, new Rectangle (0, height / 2, width, height / 2));
    }

With that in place, we get this wonderfully useless "game."

<a href="/wp-content/uploads/2015/11/ios-cocossharp-game-listview-hybrid.png"><img src="/wp-content/uploads/2015/11/ios-cocossharp-game-listview-hybrid-169x300.png" alt="iOS screenshot of a hybrid screen showing a Xamarin.Forms ListView and a CocosSharp game splitting the screen." width="169" height="300" class="alignnone size-medium wp-image-489" /></a>

<a href="/wp-content/uploads/2015/11/android-cocossharp-game-listview-hybrid.png"><img src="/wp-content/uploads/2015/11/android-cocossharp-game-listview-hybrid-180x300.png" alt="Android screenshot of a hybrid screen showing a Xamarin.Forms ListView and a CocosSharp game splitting the screen." width="180" height="300" class="alignnone size-medium wp-image-486" /></a>
