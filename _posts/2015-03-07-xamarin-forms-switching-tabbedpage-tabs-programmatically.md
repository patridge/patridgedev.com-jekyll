---
layout: post
title: "Xamarin.Forms: Switching TabbedPage Tabs Programmatically"
author: "Adam Patridge"
date: Sat, 07 Mar 2015 15:43:17 +0000
tags: Xamarin.Forms Xamarin.Android Xamarin.iOS
category: dev
excerpt_separator: <!--more-->
---

<div style="float: right; padding-left: 10px;"><a href="/wp-content/uploads/2015/03/SwitchAndPush.gif"><img src="/wp-content/uploads/2015/03/SwitchAndPush-199x300.gif" alt="iOS Switch Tabs and Push Page via button click" style="width: 200px;" class="alignnone size-medium wp-image-447" /></a></div>

Sometimes, your app uses tabs to separate different user activities. Sometimes, though, those activities aren't completely independent. When that happens, you need an action in one tab to cause the user to switch to another tab. I'll show you how to do that in code using Xamarin.Forms. In case you haven't already started your tabs, we'll start from the beginning. (Note: while the UI parts can be done quite well in XAML, this post will do it all in code.)

<div style="clear:both;" />

<!--more-->

## First, Some Tabs

Just in case you haven't already made your Xamarin.Forms tab system, we'll start there. This is as simple as using an instance of `TabbedPage`. Since we will want a little more control over things going forward, let's create a subclass inheriting from it.

    public class MainTabbedPage : TabbedPage {
        public MainTabbedPage() {
            Children.Add(new somePage());
            Children.Add(new someOtherPage());
        }
    }

You simply make this the `MainPage` in your App, now, and you get tabs.

    public class App : Application {
        public App() {
            MainPage = new MainTabbedPage();
        }
    }

If you run your app at this point, though, you'll notice something fairly obvious missing from your tabs.

<a href="/wp-content/uploads/2015/03/NoTabNames-Android.png"><img src="/wp-content/uploads/2015/03/NoTabNames-Android-180x300.png" alt="Android TabbedPage without a name" width="180" height="300" class="alignnone size-medium wp-image-441" /></a>

No titles, no icons. In fact, on iOS, you can't even tell where one tab ends and another begins.

<a href="/wp-content/uploads/2015/03/NoTabNames-iOS.png"><img src="/wp-content/uploads/2015/03/NoTabNames-iOS-200x300.png" alt="iOS TabbedPage without a name or icon" width="200" height="300" class="alignnone size-medium wp-image-439" /></a>

## Tabs UI Details

Xamarin.Forms will convert the `Title` and `Icon` properties of its child pages into these tab properties. Let's add those.

    public class MainTabbedPage : TabbedPage {
        public MainTabbedPage() {
            Children.Add(new somePage() { Title = "Tab 1", Icon = "SomeIcon1.png" });
            Children.Add(new someOtherPage() { Title = "Tab 2", Icon = "SomeIcon2.png" });
        }
    }


<a href="/wp-content/uploads/2015/03/TabsUI-Android.png"><img src="/wp-content/uploads/2015/03/TabsUI-Android-300x77.png" alt="Android Tabs UI" width="300" height="77" class="alignnone size-medium wp-image-443" /></a>

<a href="/wp-content/uploads/2015/03/TabsUI-iOS.png"><img src="/wp-content/uploads/2015/03/TabsUI-iOS-300x57.png" alt="iOS Tabs UI" width="300" height="57" class="alignnone size-medium wp-image-444" /></a>

The `Icon` property is of type FileImageSource. You can create one using the handy implicit conversion from a string as we did in the example above or create one manually.

    somePage.Icon = ImageSource.FromFile("SomeIcon1.png");
A FileImageSource will use a platform's native high-resolution image system (e.g., @2x/@3x for iOS, hdpi/xhdpi/xxhdpi for Android). For more information about using image resources in Xamarin.Forms, check out the handy [image guide on the Xamarin Developer Portal](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/working-with/images/).

## Switching Tabs By Index

Your users can always switch tabs by tapping them. To switch the selected tab programmatically, you simply change the `CurrentPage` property of your TabbedPage.

    CurrentPage = Children[1];

Without a reference to your child pages, you would be stuck using the index based on the order they were added. We could certainly do that if we want to hard-code what each index translates to. For instance, we could set it up to confuse our user by randomly switching tabs from the constructor. (Don't do this.)

    // NOTE: please don't randomly switch tabs on your user for fun.
    Task.Run(async () => {
        var nextChildIndex = 0;
        while (true) {
            nextChildIndex = nextChildIndex == 0 ? 1 : 0;
            await Task.Delay(3000);
            Device.BeginInvokeOnMainThread(() => {
                CurrentPage = Children[nextChildIndex];
            });
        }
    });

Not that this deserves analysis, but this is not terribly complex (which is ideal for a mock example). We start up an asynchronous Task that constantly waits three seconds before switching to the next tab (index 0 or index 1). We also make sure to swap the current page on the UI thread. Even though it looks like an loop that would lock up the app, the async/await means it is never waiting on the UI thread.

## Switching Tabs By Reference
To be able to switch tabs by reference, let's maintain the child pages in fields. Now, with a few switch functions, we can end up on the desired tab without worrying about indexes. At some point later, if you want to change the order of the tabs, you only need to change the order they are added to `Children`.

    public class MainTabbedPage : TabbedPage {
        readonly Page tab1Page;
        readonly Page tab2Page;
    
        public MainTabbedPage() {
            tab1Page = new Page1() { Title = "Tab 1 title", Icon="circle.png" };
            tab2Page =new Page2() { Title = "Tab 2 title", Icon="square.png" };

            // To change tab order, just shuffle these Add calls around.
            Children.Add(tab1Page);
            Children.Add(tab2Page);
        }
    
        public void SwitchToTab1() {
            CurrentPage = tab1Page;
        }
        public void SwitchToTab2() {
            CurrentPage = tab2Page;
        }
    }

From within a tabbed page, you can now call to these methods by way of the Parent property.

    public class Page1 : ContentPage {
        public Page1() {
            var button = new Button() {
                Text = "Switch to Tab 2",
            };
            button.Clicked += async (sender, e) => {
                var tabbedPage = this.Parent as TabbedPage;
                tabbedPage.SwitchToTab2();
            };
            Content = new StackLayout { 
                Children = {
                    button,
                }
            };
        }
    }

In this case, we know that Page1 will only exist within a TabbedPage, but you could also exclude that code, or guard the switch call, with a conditional check to make sure you `Parent is TabbedPage`.

## Switch and Push?

Now that you have a reference to each tab's Page, you can also do a little work on the way there during a switch. For example, if your destination tab child is a NavigationPage, you could push a new page on its stack. In our prior example, we could accept a destination Page as a parameter on `SwitchToTab1`.

    readonly NavigationPage tab1Page;
    // ...
    public async Task SwitchToTab1(Page pageToPush) {
        CurrentPage = tab1Page;
        if (pageToPush != null) {
            await tab1Page.PushAsync(pageToPush);
        }
    }

It may not have been obvious, but the signature of our switch function has changed from above. Since the `PushAsync` method is asynchronous, we should await it so any calling code can await this call as well. (Note: `async void` is potentially dangerous, so we give it a Task return type. For more background on this, check out this [video by async-master Lucian Wischik's](http://channel9.msdn.com/Series/Three-Essential-Tips-for-Async/Tip-1-Async-void-is-for-top-level-event-handlers-only.)

<a href="/wp-content/uploads/2015/03/SwitchAndPush.gif"><img src="/wp-content/uploads/2015/03/SwitchAndPush-199x300.gif" alt="iOS Switch Tabs and Push Page via button click" width="199" height="300" class="alignnone size-medium wp-image-447" /></a>

## Bonus Quirk: Android With Tabs + Navigation

If you have used Xamarin Forms enough, you know that things are rarely one-to-one between platforms. In this case, if you make a NavigationPage a child of a TabbedPage, it will look a little counter-intuitive on Android. The navigation is done first as the ActionBar and the ActionBar tabs sit directly underneath that bar, almost the opposite of the structure you created. This seems to be the [expected representation of tabs in Android](http://developer.android.com/design/building-blocks/tabs.html), so there may not be a workaround; and if there is, it may confuse Android users expecting the typical layout.

## Sample Code

To see what would barely be called "finished code", [check out my Xamarin Forms TabbedPage GitHub repo](https://github.com/patridge/XamarinForms-Tabs-Demo) of what I was working on while writing this article.
