---
layout: post
title: "Where did that JSON field go? Serializing IHtmlString to JSON"
author: "Adam Patridge"
date: Thu, 05 Jul 2012 18:26:17 +0000
tags: asp-net-mvc c-sharp json ihtmlstring
category: dev
excerpt_separator: <!--more-->
---

## TL;DR

If your brain consumes Stack Overflow questions better than blog posts, go see ["How do I serialize IHtmlString to JSON with Json.NET?"](http://stackoverflow.com/q/11350392/48700) over there.

<!--more-->

## IHtmlString doesn't play nicely with JSON serialization

If you have an IHtmlString in your JSON (regardless of arguments against putting raw HTML in JSON), you will probably need to customize the serialization to get the HTML out of that variable. In fact, the serialization will probably be invalid compared to what you expect; it does make sense if you think about how the serialization process works.

Fortunately, putting together a quick Json.NET `JsonConverter` will make the issue go away.

What you might expect from some object containing an `IHtmlString` variable named `x`:

    { x = "some <span>text</span>" }

What default .NET JavaScript serialization and default Json.NET serialization will give you for said object:

    { x = { } }

How to fix it with Json.NET:

    public class IHtmlStringConverter : Newtonsoft.Json.JsonConverter {
        public override bool CanConvert(Type objectType) {
            return typeof(IHtmlString).IsAssignableFrom(objectType);
        }
        ...
        public override void WriteJson(Newtonsoft.Json.JsonWriter writer, object value, Newtonsoft.Json.JsonSerializer serializer) {
            IHtmlString source = value as IHtmlString;
            if (source == null) {
                return;
            }
            writer.WriteValue(source.ToString());
        }
    }

## Background

While working on [some random API](http://dev.sierratradingpost.com/), we noticed one of our JSON fields went from being a long string containing raw HTML (legacy baggage) to an empty object: `SomeFieldWithHtml: {}`.

While the API JSON-generating code hadn't changed, that field was pulled from code shared by our MVC website. It seems that field was converted directly to an IHtmlString to avoid doubly-encoding it in an MVC view. If you ever have the same issue, you can avoid this issue entirely by leaving the source as a string and doing a quick `MvcHtmlString.Create(x)` on it before sending it to your view/view-model.

## Code

For the full source used in this post, including an example MVC project where a controller has action methods to test all the variations here, head over to [this post's GitHub repository](https://github.com/patridge/JsonNetIHtmlStringTesting).

---

## Comments

* **Bryan**, _2013-01-23 19:35:23 +0000_

    > This was very helpful, thanks! I was trying to pass a single-line html string to a ckeditor instance. I ended up just removing the line breaks and such and that seemed to do the trick.

