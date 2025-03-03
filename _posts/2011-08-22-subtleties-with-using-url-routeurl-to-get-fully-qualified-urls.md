---
layout: post
title: "Subtleties with using Url.RouteUrl to get fully-qualified URLs"
author: "Adam Patridge"
date: Tue, 23 Aug 2011 04:34:46 +0000
tags: c-sharp asp-net-mvc
category: dev
excerpt_separator: <!--more-->
---

At some point I missed the Url.RouteUrl overload that took a protocol and returned an absolute URL based on the current context. It is quite handy when you are sending URLs out into the world (e.g., RSS feed link). I ended up using the less-handy overload that took an explicit host (same as the current, in this case) and passing it in. When someone pointed out the simpler overload, I did the obvious and deleted the host from the call. That didn't quite work.

For those looking for a way to get a fully-qualified URL through the route system, the less-than-obvious answer is to call the overload for Url.RouteUrl that [gets a URL with a different protocol](http://msdn.microsoft.com/en-us/library/dd492238.aspx) (`Url.RouteUrl(string, object, string)`), passing in `Request.Url.Scheme` for the current protocol.

<!--more-->

    Url.RouteUrl("Default", new { action = "Index", controller = "Department", id = 1 }, Request.Url.Scheme);
    // http://www.currentdomain.com/department/index/1

Say you want to send someone to a different subdomain in your app while using the same routes. There's an [overload for that](http://msdn.microsoft.com/en-us/library/dd460200.aspx): `Url.RouteUrl(string, RouteValueDictionary, string, string)`. Combined with the above example, here's how these all play out if you are currently handling a www.currentdomain.com request and your route table includes the fallback default (`{controller}/{action}/{id}`).

    Url.RouteUrl("Default", new { action = "Index", controller = "Department", id = 1 });
    // /department/index/1
    Url.RouteUrl("Default", new { action = "Index", controller = "Department", id = 1 }, Request.Url.Scheme);
    // http://www.currentdomain.com/department/index/1
    Url.RouteUrl("Default", new RouteValueDictionary(new { action = "Index", controller = "Department", id = 1 }), "http", "sub.currentdomain.com");
    // http://sub.currentdomain.com/department/index/1

Now if you switch between the two fully-qualified calls, you may try just deleting or adding the `hostName` parameter, respectively. One direction is a compile error, and one direction is a runtime oddity resulting in a hideous URL.

    Url.RouteUrl("Default", new { action = "Index", controller = "Department", id = 1 }, "http", "sub.currentdomain.com");
    // Compile error (expects a RouteValueDictionary)
    Url.RouteUrl("Default", new RouteValueDictionary(new { action = "Index", controller = "Department", id = 1 }), "http", "sub.currentdomain.com");
    // Eye-bleeding and incorrect route as it "serializes" the RouteValueDictionary.
    // In my case, I ended up with something like this:
    // http://www.currentdomain.com/current/route/1/?Count=3&Keys=System.Collections.Generic.Dictionary%602%2BKeyCollection%5BSystem.String%2CSystem.Object%5D&Values=System.Collections.Generic.Dictionary%602%2BValueCollection%5BSystem.String%2CSystem.Object%5D
    
On a side note, if your development environment uses localhost with a port and you use some web.config app setting for that URL change between development and production ("localhost:12345" vs "www.currentdomain.com"). You will want your host setting to be without the port. `Url.RouteUrl` will hiccup on your development environment if the port is part of the host name (it's no longer just the host at that point).

    Url.RouteUrl("Default", new RouteValueDictionary(new { action = "Index", controller = "Department", id = 1 }), "http", ConfigurationManager.AppSettings["hostwithport"]);
    // http://localhost:12345:12345/department/index/1

---

## Comments

* **Richard Bogle**, _2013-03-22 16:33:33 +0000_

    > These peculiar results were causing me a lot of grief trying to get RouteUrl working. Thanks to your post I was able to try a couple more things to get my work done.

* **Gary**, _2014-04-06 20:26:50 +0000_

    > I was using Url.Action (I'm a noob) with similar route values and it ended clashing with one of my create views. Found Url.RouteUrl and stumbled onto this, working perfectly; great resource! Thank you!

* **Radu Bartan**, _2019-06-01 01:19:06 +0000_

    > Adam,
    >
    > I've been trying like hell to get some sort of link to work nice with a route name inside an area. Do you have any suggestions?