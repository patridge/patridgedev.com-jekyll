---
layout: post
title: "Creating an animated spinner in a Xamarin.iOS (MonoTouch) UIImageView"
date: Wed, 18 Jul 2012 14:39:04 +0000
tags: Xamarin UIImageView Xamarin.iOS MonoTouch
# category: dev
excerpt_separator: <!--more-->
---

If you ever poke around routing in your MVC app's Global.asax code, you'll probably see a call like this from the initial project creation:

    public static void RegisterRoutes(RouteCollection routes) {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
        ...
    }

If you ever decide to add your own ignored route, perhaps following the [example from Phil Haack for favicon.ico requests](http://haacked.com/archive/2008/07/14/make-routing-ignore-requests-for-a-file-extension.aspx), you will probably see two methods on that `RouteCollection` object that seem quite similar: `routes.Ignore` (a method on `RouteCollection`) and `routes.IgnoreRoute` (an extension method on `RouteCollection` from the `System.Web.Mvc` namespace).
This is a call to tell the routing system not to bother processing any requests for files at the root level that have a ".axd" in them. URLs that skip routing because of `routes.IgnoreRoute("{resource}.axd/{*pathInfo}");`:

<!--more-->

### Sources

* [ASP.NET web stack source for `System.Web.Mvc.RouteCollectionExtensions.cs`](http://aspnetwebstack.codeplex.com/SourceControl/changeset/view/44b53cd4dd84#src%2fSystem.Web.Mvc%2fRouteCollectionExtensions.cs)
* [Phil Haack's seminal post on `IgnoreRoute`](http://haacked.com/archive/2008/07/14/make-routing-ignore-requests-for-a-file-extension.aspx).
* [MSDN docs on RouteCollection.Ignore method](http://msdn.microsoft.com/en-us/library/dd992982.aspx)
* [MSDN docs on RouteCollection.IgnoreRoute extension method](http://msdn.microsoft.com/en-us/library/dd470170.aspx)
