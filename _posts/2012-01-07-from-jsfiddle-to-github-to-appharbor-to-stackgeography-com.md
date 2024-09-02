---
layout: post
title: "From jsFiddle to GitHub to AppHarbor to stackgeography.com"
date: Sun, 08 Jan 2012 05:33:53 +0000
tags: API JavaScript Stack-Exchange
# category: dev
excerpt_separator: <!--more-->
---

### Update

I finally put together an app listing on [StackApps](http://stackapps.com). If you like [stackgeography.com](http://www.stackgeography.com/) and you have a StackApps account, I'd love for your vote on the [StackGeography app page](http://stackapps.com/questions/2913/stackgeography-a-stack-exchange-question-mapping-site).

### Background

My latest project at my full-time job with [Sierra Trading Post](http://www.sierratradingpost.com) is their [Mashery-managed API](http://dev.sierratradingpost.com). It currently includes most everything you would expect from a retail API: departments, brands, products, search, and reviews. I would like it to include a few things that are less typical but fun to work with from a data aggregation standpoint. One example: recent order locations. While it may not see a lot of outside use, it would be quite nice for our internal observation; it would go great on the handful of large screens around the company with site statistics continually updating on them.

<!--more-->

In order to put together a proof-of-concept that I could even do this, I wanted to make a mapping demo using someone else's API. Since I am responsible for the Sierra Trading Post API, I try to research other APIs to learn from their successes and pain points. One such API that has provided a lot of [positive] learning is the [Stack Exchange API](http://api.stackexchange.com/docs/). It just so happened that they were releasing version 2 of their API, so I decided to use that for my first proof-of-concept mapping fun. It didn't hurt that they were [tossing out a few prizes](http://blog.stackoverflow.com/2011/12/stack-exchange-api-v2-0-public-beta/) over on [StackApps.com](http://stackapps.com) for good apps using the updated API.

I started with a few minutes of poking around the [Google Maps API](http://code.google.com/apis/maps/documentation/javascript/basics.html) and quite a few more minutes poking around the 2.0 version of the [Stack Exchange API](http://api.stackexchange.com/docs/). After that, I immediately ran to my favorite HTML/JavaScript/CSS prototyping grounds.

### Stage 1: jsFiddle

It all started in [jsFiddle](http://jsfiddle.net) and grew into a huge mess of nasty JavaScript. After exactly 337 revisions over a week, some to get the basics working and some to refactor the ugliness, I decided to put the code and markup somewhere more permanent. Along the way, I learned a few things.

* When you poll an API regularly, you eat up your daily allocation of requests pretty quickly (300 per IP per day for the Stack Exchange API), even when you bump it back to every minute and have multiple tabs running simultaneously.
* APIs never offer things exactly the way you want it. It certainly makes me sympathetic to the API users I cater to.
* For any fairly complex JavaScript, you run out of room on a single screen with jsFiddle's four iframes, Chrome's JavaScript debugging window, and Chrome's debugging console on a single screen.

### Stage 2: GitHub

After exactly 337 revisions over a week through jsFiddle, I decided to put the code and markup somewhere more permanent. Enter GitHub and the [StackGeography repository](https://github.com/patridge/StackGeography). I've played with Git about 20 minutes total, but I'd researched it for several hours and I have played with Mercurial enough to understand most of the core concepts. Since it was already installed and configured for another project, I was up and running in a few minutes.

I copied out the JavaScript and CSS into separate files, and threw the HTML into an HTML5 page referencing them both. From [WebMatrix 2 Beta](http://www.microsoft.com/web/webmatrix/next/), I opened the folder as a new site. I was now working from a local version of my jsFiddle. Of course, my hosting options were limited. I have a tendency to "host" files from my [DropBox's](http://db.tt/MM5CStl) public folder, especially files I reference in jsFiddle demos. I could also slap it up on one of my other domains, I even considered this one, but I wanted something more professional.

### Stage 3: StackGeography.com + AppHarbor

After I ruled out hosting under an existing domain, I knew I wanted the project to have its own domain. I had already named the project "StackGeography" (formerly "stackviz") on its StackApps registration, so I gave that a shot and found [stackgeography.com](http://www.stackgeography.com/) available. I know everyone hates GoDaddy for a bunch of things (and I totally agree on the [SOPA](http://en.wikipedia.org/wiki/Stop_Online_Piracy_Act) issue), but they are dirt cheap for domain registrations. Despite the terrible experience I had with their 4G hosting off this WordPress blog, I haven't had any trouble with domain registrations or their name servers. After tip-toeing through the upsell landmines, I had the domain. Having a domain, though, didn't get me hosting. 

Somewhere in my past, I had read a few snippets on [AppHarbor](https://appharbor.com/) and how it could automagically publish your ASP.NET projects from a source control repository. Since I tend to hate all the overhead that goes into setting up those things manually, and I wanted to be quick on this one, I spent some time researching their system. It seemed simple enough, and it really is; you just configure the AppHarbor GitHub service hook. Sadly, it took me three blog posts about their system to figure out what they meant by "Application Slug"; it's just the name of your AppHarbor application. I've heard the term "slug" used that way before, but it just didn't click for me here.

Of course, I had to change up my project a little. Since AppHarbor expects a .NET project, I moved all my files to an new, empty ASP.NET application and committed the changes to GitHub. In a few minutes, I checked [stackgeography.com](http://www.stackgeography.com/) and found my demo working fine, it was probably there in a few seconds since updates seemed to be almost instant. Amazing stuff.

*Update 2012-02-03: AppHarbor was apparently in beta this whole time and I didn't notice. They have since announced a pricing system, including charging for custom hostnames; you still get a free {yourappname}.apphb.com subdomain to use. Since stackgeography.com is not a revenue-generating site (nor do I plan to make it one), I will probably just use their free one and point stackgeography.com at it.*

### Check it out

While I can't guarantee the app won't blow through its API allocation of 10,000 requests per day (at 300 per IP), I would love to have you [check out stackgeography.com](http://www.stackgeography.com/) and give me some feedback about it. Bugs, features, or otherwise, it's out there for anyone to see it working or see how it works. I have plenty of plans on features to add, and I definitely want to improve the code quite a bit before I put it to use with the Sierra Trading Post API, but I think now is as good a time as any to get it out in the hands of others.

If you want to learn more about my plans for [stackgeography.com](http://www.stackgeography.com/), check out the [StackGeography README.md file on GitHub](https://github.com/patridge/StackGeography).
