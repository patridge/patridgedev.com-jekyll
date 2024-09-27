---
layout: post
title: "Handling case-insensitive enum action method parameters in ASP.NET MVC"
author: "Adam Patridge"
date: Fri, 19 Aug 2011 03:00:29 +0000
tags: C# ASP.NET-MVC
category: dev
excerpt_separator: <!--more-->
---

[Skip to solution](#solution-handling-case-insensitive-enum-action-method-parameters)

## Using enums as action method parameters

Say you have a enum, a sorting enum in this case.

    public enum SortType {
        NewestFirst,
        OldestFirst,
        HighestRated,
        MostReviews
    }

ASP.NET will gladly let you use that enum as an action method parameter; you can even make it an optional parameter. To make it optional by routing, you need to make it nullable for the action method function parameter in your controller and add some guarding logic (!sort.HasValue or the like).

<!--more-->

    routes.MapRoute(
        "DepartmentProducts",
        "Department/Products/{sort}",
        new { controller = "Department", action = "Products", sort = UrlParameter.Optional }
    );
    public ActionResult Products(SortType? sort) {
        SortType requestedSort = sort ?? SortType.NewestFirst;
        ...
    }

To make the parameter optional by function parameter, just give the parameter a default value.

    routes.MapRoute(
        "DepartmentProducts",
        "Department/Products/{sort}",
        new { controller = "Department", action = "Products" }
    );
    public ActionResult Products(SortType sort = SortType.NewestFirst) { ... }

Both work just fine, though I lean toward the function parameter default. Regardless of implementation, you can call the method by a number of different URLs:

* http://www.somedomain.com/department/products/ (sort == null or SortType.NewestFirst, respectively)
* http://www.somedomain.com/department/products/OldestFirst/
* http://www.somedomain.com/department/products/HighestRated/
* http://www.somedomain.com/department/products/?sort=MostReviews

Unfortunately, it requires the route value to be an exact match for the enum name, proper case included. This URL will not result in the correct value for the sort parameter.

    http://www.somedomain.com/department/products/oldestfirst (results in null on the former, NewestFirst on the later)

This can be a problem with the internet being mostly case-insensitive, especially if you have a [lower case routing system](http://stackoverflow.com/questions/878578/how-can-i-have-lowercase-routes-in-asp-net-mvc/931764#931764) and/or use the [lowercase SEO tweak](http://weblogs.asp.net/scottgu/archive/2010/04/20/tip-trick-fix-common-seo-problems-using-the-url-rewrite-extension.aspx) in the [IIS URL rewrite](http://www.iis.net/download/urlrewrite) system (future post coming about a gotcha and work-around on applying this handy system to existing sites).

## Solution {#solution-handling-case-insensitive-enum-action-method-parameters}

While looking into this, I came across just the code I thought I needed from [Rupert Bates](http://eliasbland.wordpress.com/2009/08/08/enumeration-model-binder-for-asp-net-mvc/). It is case-insensitive model binder that is designed to allow you to declare a site-wide default for a given enum type. This was a great starting point. Since I tend to use the optional function parameters, though, this actually meant the model binder would give me a default before the action method could even try to do the same. I pulled that part from the version I used.

I tied the generic, as best as possible, to an enum (`where T : struct`) so it is harder to use it for types that aren't compatible. While I was poking around, I flipped it to use a once-built look-up table for the enum using a case-insensitive `Dictionary<string, T>` rather than repeat calls to Enum.Parse (called once per value) and Enum.GetNames (called only once). As far as my manual testing has shown me, it works quite well and should be at least slightly faster, not that Rupert's solution couldn't hold its own just fine.

    using System;
    using System.Web.Mvc;
    using System.Collections.Generic;

    namespace StpWeb.CustomModelBinders {
        public class EnumBinderIgnoreCase<T> : IModelBinder where T : struct {
            public EnumBinderIgnoreCase() {
                foreach (string enumName in enumNames) {
                    enumLookups.Add(enumName, (T)Enum.Parse(typeof(T), enumName, true));
                }
            }
            public object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext) {
                return bindingContext.ValueProvider.GetValue(bindingContext.ModelName) == null
                    ? (T?)null
                    : GetEnumValue(bindingContext.ValueProvider.GetValue(bindingContext.ModelName).AttemptedValue);
            }

            private string[] enumNames = Enum.GetNames(typeof(T));
            private Dictionary<string, T> enumLookups = new Dictionary<string, T>(StringComparer.OrdinalIgnoreCase);
            private T GetEnumValue(string value) {
                T foundEnumValue = default(T);
                if (!String.IsNullOrEmpty(value) && enumLookups.ContainsKey(value))
                    foundEnumValue = enumLookups[value];

                return foundEnumValue;
            }
        }
    }
