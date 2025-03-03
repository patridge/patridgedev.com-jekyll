---
layout: post
title: "Initial setup for a new ASP.NET MVC site in IIS7"
author: "Adam Patridge"
date: Fri, 27 Jan 2012 17:26:56 +0000
tags: asp-net-mvc iis json
category: dev
excerpt_separator: <!--more-->
---

## Background

Over the years, I have spent far too many hours running the same set of commands against ASP.NET and ASP.NET MVC sites to bring them up to what I consider a starting point. Most of the time, I have to refresh myself about how to do at least one of them. This is a list of those commands in a central location for myself and anyone else to use. As with any good instructions, there is no guarantee you won't completely destroy your server, site, or soul using these commands. I can only say they worked for me once.

<!--more-->

1. [Remove unnecessary HTTP headers](#remove-headers-root-initial-asp-net-site-iis-setup)
  1. [Server](#remove-headers-server-initial-asp-net-site-iis-setup)
  1. [X-Powered-By](#remove-headers-x-powered-by-initial-asp-net-site-iis-setup)
  1. [X-AspNet-Version](#remove-headers-x-aspnet-version-initial-asp-net-site-iis-setup)
  1. [X-AspNetMvc-Version](#remove-headers-x-aspnetmvc-version-initial-asp-net-site-iis-setup)
1. [Adding dynamic content compression (e.g., GZIP)](#adding-gzip-initial-asp-net-site-iis-setup)
1. [GZIP for JSON and JSONP content types (e.g., "application/json")](#gzip-for-json-initial-asp-net-site-iis-setup)
1. [Complete appcmd command-line list](#command-line-list-initial-asp-net-site-iis-setup)

## Remove unnecessary HTTP headers {#remove-headers-root-initial-asp-net-site-iis-setup}

By default, IIS and ASP.NET give you a few headers in each and every HTTP response sent out.

    Server: Microsoft-IIS/7.5 (or 7.0, or whatever IIS version you have)
    X-Powered-By: ASP.NET
    X-AspNet-Version: 4.0.303319 (or 2.0.50727, or 1.1.4322, or whatever version you have)
    X-AspNetMvc-Version: 3.0 (or 2.0, or 1.0, or whatever version you have)

People will [rant and rave](http://forums.iis.net/p/1171860/1967158.aspx#1967158) about removing these headers for security and whether that helps. I am firmly on the side of "not-why-I-care-at-all". I always figure if I am a target, obscurity doesn't buy me anything when someone with very little computing power can just throw packets exploiting all the major frameworks at me and see which stick (which happens even when you aren't a real "target"). The reason I do care is that removing these headers strips out even more data that has to be sent between you and the end user. Even though the size difference is small, it applies to every single HTTP response handled by your site. For such simple one-time changes, I will gladly take the savings.

### Server (the annoying one) {#remove-headers-server-initial-asp-net-site-iis-setup}

This HTTP header appears to be the most annoying to remove. In fact, short of an HTTP module, the common method for doing so is a new line in a [probably new] event handler in global.asax.cs (or global.asax.vb).

    protected void Application_PreSendRequestHeaders() {
        HttpContext.Current.Response.Headers.Remove("Server");
    }
    
It's not as pretty as the rest of the removal stuff below, but it gets the job done. In fact, they could all be solved this way, but I tend to lean on IIS to do the heavy lifting when I can.

### X-Powered-By (the more annoying web.config/IIS one) {#remove-headers-x-powered-by-initial-asp-net-site-iis-setup}

Many places that offer suggestions on removing the `X-Powered-By: ASP.NET` header talk about using IIS. You can certainly do this if you have permission to do so on your hosting provider and like the IIS GUI. I prefer things that I can push out with source control, so I go the web.config route.

    <system.webServer>
        <httpProtocol>
            <customHeaders>
                <remove name="X-Powered-By" />
            </customHeaders>
        </httpProtocol>
    </system.webServer>
    
Command line happiness:

    %windir%\System32\inetsrv\appcmd.exe set config "{your-site-name}" /section:system.webServer/httpProtocol /-"customHeaders.[name='X-Powered-By']"
    
The root source of this setting is actually found in your server's applicationHost.config (`%windir%\system32\inetsrv\config\applicationHost.config`). If you have the ability to modify that file, you can remove it there to make it the default for all IIS sites.

    <system.webServer>
        <httpProtocol>
            <customHeaders>
                <clear />
                <add name="X-Powered-By" value="ASP.NET" /> <!--Remove this line.-->
            </customHeaders>
            <redirectHeaders>
                <clear />
            </redirectHeaders>
        </httpProtocol>
    </system.webServer>
    
Command line happiness:

    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpProtocol /-"customHeaders.[name='X-Powered-By']" /commit:apphost

### X-AspNet-Version (the easy web.config one) {#remove-headers-x-aspnet-version-initial-asp-net-site-iis-setup}

To make `X-AspNet-Version: {your-aspnet-version}` go away, you just need to tweak your site's web.config file. In the `<system.web>` section, under `<httpRuntime>`, which is probably already there, just make sure the `enableVersionHeader="false"` attribute is added to it.

    <system.web>
        <httpRuntime enableVersionHeader="false" />
    </system.web>

Command line happiness:
    
    %windir%\System32\inetsrv\appcmd.exe set config "{your-site-name}" /section:system.web/httpRuntime /enableVersionHeader:false

I'm guessing this could be done at the applicationHost.config level, though I have never tried it (no real excuse, just too scared to mess with such a vital config entry too much).
    
### X-AspNetMvc-Version (the global.asax code one) {#remove-headers-x-aspnetmvc-version-initial-asp-net-site-iis-setup}

Obviously, if you are tweaking a non-MVC ASP.NET site, you can ignore this one.

To make `X-AspNetMvc-Version: {your-aspnetmvc-version}` go away, you need to add some code and redeploy your site. In your global.asax.cs (or global.asax.vb) file, you will need to add a single line to the application start event.

    protected void Application_Start() {
        MvcHandler.DisableMvcResponseHeader = true;
        // ...
    }

## Adding dynamic content compression (e.g., GZIP) {#adding-gzip-initial-asp-net-site-iis-setup}

Reducing the amount of data you send over the wire is pretty good idea, especially if you are targeting low-bandwidth users like mobile browsers. While GZIP has some performance impact on the server and client, it is negligible on most systems. To add this system to IIS, add in the "Dynamic Content Compression" role service under the "Web Server (IIS)" role in "Server Manager". If I am doing it visually, I typically just follow along with this [iis.net article on the process](http://www.iis.net/ConfigReference/system.webServer/httpCompression#003).

Command line happiness:

    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/globalModules /+[name='DynamicCompressionModule',image='%windir%\System32\inetsrv\compdyn.dll'] /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/modules /+[name='DynamicCompressionModule',lockItem='true'] /commit:apphost
    
If this has already been installed, this will error out with a message about adding a duplication "DynamicCompressionModule" entry.

## GZIP for JSON and JSONP content types (e.g., "application/json") {#gzip-for-json-initial-asp-net-site-iis-setup}

*[Update 2012-02-02] NOTE: If you have a load-balancer, you may need to figure out how to get it to allow these content types through as GZIP. In one situation where I added this setting to a series of servers behind F5's BIG-IP load balancer, it still wouldn't get GZIP out to the requester. BIG-IP was either stripping the request of `Accept-Encoding: gzip` or getting GZIP from IIS and decompressing it before letting it return to the original requester (I didn't confirm what was happening, but I would assume the former since it is less CPU-intensive).*

While getting GZIP compression is great for HTML, JavaScript, CSS, and most others with "Dynamic Content Compression" is great, it drops the ball on content types it doesn't have explicit orders to manipulate. One such content type that should get compression is dynamically-generated JSON and JSONP.

While working on the [Sierra Trading Post API](http://dev.sierratradingpost.com/] I noticed that IIS with dynamic compression enabled doesn't do anything to JSON content types, "application/json" (JSON) and "application/javascript" (JSONP). Compression is decided by the content types listed in applicationHost.config (`%windir%\system32\inetsrv\config\applicationHost.config`). By default, you get most of the types served by the average ASP.NET site: `text/*` (covering `text/html`). If you need to add more to this list, add them to the config file. It does not matter if they come before or after the `<add mimeType="*/*" enabled="false" />` wildcard entry that is there by default.

    <system.webServer>
        <httpCompression ... >
            ...
            <dynamicTypes>
                ...
                <add mimeType="application/json; charset=utf-8" enabled="true" />
                <add mimeType="application/javascript; charset=utf-8" enabled="true" />
            </dynamicTypes>
        </httpCompression>
    </system.webServer>
    
Command line happiness:
    
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/javascript; charset=utf-8',enabled='True']" /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost
    
For some reason, these are not recognized without the `charset=utf-8` part. As well, if you don't notice a change right away, that's normal; you must recycle the app pool first (TechNet has a [nice article with info about app pool recycling](http://technet.microsoft.com/en-us/library/cc745955.aspx) and why it doesn't happen here).

     %windir%\System32\inetsrv\appcmd recycle apppool /apppool.name:"{your-app-pool-name}"

Again, some people would suggest an HTTP module for handling this, but it seems like IIS would be better equiped to handle this since it is already doing so for the rest of my outbound content.

### Complete appcmd command-line happiness {#command-line-list-initial-asp-net-site-iis-setup}

If you are following along at home, most of this stuff can be poked into server config files, either applicationHost.config or your site's web.config (all but the `Server: Microsoft-IIS/7.5` HTTP header). If you want the fire-and-forget solution, here are all the commands smashed together for your batch file use.

    %windir%\System32\inetsrv\appcmd.exe set config "{your-site-name}" /section:system.webServer/httpProtocol /-"customHeaders.[name='X-Powered-By']"
    %windir%\System32\inetsrv\appcmd.exe set config "{your-site-name}" /section:system.web/httpRuntime /enableVersionHeader:false
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/globalModules /+[name='DynamicCompressionModule',image='%windir%\System32\inetsrv\compdyn.dll'] /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/modules /+[name='DynamicCompressionModule',lockItem='true'] /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/javascript; charset=utf-8',enabled='True']" /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost
    %windir%\System32\inetsrv\appcmd recycle apppool /apppool.name:"{your-app-pool-name}"

If you want to do all this to applicationHost.config alone, you just swap out the first two lines from hitting "{your-site-name}" to pointing to the apphost config. Here they are all smashed together for that variant. NOTE: I have not confirmed that setting `enableVersionHeader` in applicationHost.config will work.

    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpProtocol /-"customHeaders.[name='X-Powered-By']" /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config /section:system.web/httpRuntime /enableVersionHeader:false /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/globalModules /+[name='DynamicCompressionModule',image='%windir%\System32\inetsrv\compdyn.dll'] /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/modules /+[name='DynamicCompressionModule',lockItem='true'] /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/javascript; charset=utf-8',enabled='True']" /commit:apphost
    %windir%\System32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost
    %windir%\System32\inetsrv\appcmd recycle apppool /apppool.name:"{your-app-pool-name}"

---

## Comments

* **Brett**, _2012-06-13 18:13:33 +0000_

    > What is the best way to verify that it worked? I am only adding JSON content to compression and removing the HTTP Headers?
    >
    > thanks.

    * **Adam Patridge**, _2012-06-18 04:14:37 +0000_

        > I find it easiest to verify things go across the wire as desired with a browser's developer tools. In Chrome, load up the tools and make the request again. In the dev tool's network tab, you will see your requests logged. Click the request you wish to check and look at the response headers sent back by your site. If you see something like "Content-Encoding:gzip" in the response headers, it worked fine.

* **Rich M.**, _2014-09-24 20:55:19 +0000_

    > Thanks, This helped. We were having issues where gzip was successful for xml but not for json. Our F5 is new and we had compression turned on but were not allowing json. We simply added application\json it to the compression config and it worked. Now we'll need to test and decide if it is better for us to have our webservers do the compressing or the load balancer.

* **Robert Crew**, _2015-08-12 05:29:10 +0000_

    > Thanks for the tutorial.  I've found that enableVersionHeader does not work when committed to applicationHost.config.  The AppPool closes with an error as soon as it starts.
    >
    > The line I use is:
    > %windir%\System32\inetsrv\appcmd.exe set config /section:system.web/httpRuntime /enableVersionHeader:false /commit:webroot
