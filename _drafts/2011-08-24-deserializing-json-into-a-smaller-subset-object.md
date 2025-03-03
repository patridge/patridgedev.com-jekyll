---
layout: post
title: "Deserializing JSON into a smaller subset object"
author: "Adam Patridge"
date: Wed, 24 Aug 2011 22:32:40 +0000
tags: deserialization json expandoobject
category: dev
excerpt_separator: <!--more-->
---

## TODO: add Receptacle2 and test ServiceStack.Text

## Evolving JSON

If you ever serialize something into a subset object, both `System.Web.Script.Serialization.JavaScriptSerializer` and JSON.NET will handle it for you.

Say you have a receptacle class for your JSON.

    dynamic test = new ExpandoObject();
    test.x = "xvalue";
    test.y = MvcApplication.TimeStamp;

    System.Web.Script.Serialization.JavaScriptSerializer javaScriptSerializer = new System.Web.Script.Serialization.JavaScriptSerializer();
    string jsonOfTest = javaScriptSerializer.Serialize(test);
    ViewBag.Results.DefaultSerializer = jsonOfTest;

    javaScriptSerializer.RegisterConverters(new System.Web.Script.Serialization.JavaScriptConverter[] { new ExpandoJsonConverter() });
    jsonOfTest = javaScriptSerializer.Serialize(test);
    ViewBag.Results.PostCustomSerializer = jsonOfTest;
    TestReceptacle deserializedJson = javaScriptSerializer.Deserialize<TestReceptacle>(jsonOfTest);
    //dynamic deserializedJsonAsExpandoObject = javaScriptSerializer.Deserialize<ExpandoObject>(jsonOfTest);
    //ViewBag.Results.DeserializeExpandoTest = string.Format("x:{0}, y:{1}", deserializedJsonAsExpandoObject.x.ToString(), deserializedJsonAsExpandoObject.y.ToString());

    test.z = 15.434m;
    jsonOfTest = javaScriptSerializer.Serialize(test);
    ViewBag.Results.PostCustomSerializerAdditionalItem = jsonOfTest;
    deserializedJson = javaScriptSerializer.Deserialize<TestReceptacle>(jsonOfTest);
