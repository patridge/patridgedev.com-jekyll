---
layout: post
title: "Getting dynamic ExpandoObject to serialize to JSON as expected"
author: "Adam Patridge"
date: Thu, 25 Aug 2011 02:44:18 +0000
tags: C# JavaScriptSerializer JSON Json.NET ServiceStack.Text
category: dev
excerpt_separator: <!--more-->
---

## Serializing ExpandoObjects

I am currently creating a JSON API for a handful of upcoming Sierra Trading Post projects. When I found myself generating JSON for a stripped-down representation of a number of domain classes, all wrapped with some metadata, I turned to `dynamic` and things have been going quite well. Unfortunately, there was a hurdle to getting the JSON to look the way I wanted.

If you start playing around with serializing `ExpandoObject` to JSON, you will probably start finding a number of options. The easiest solution to find is the one that [comes with .NET](http://msdn.microsoft.com/en-us/library/bb302399.aspx), `JavaScriptSerializer` under `System.Web.Script.Serialization`. It will happily serialize an `ExpandoObject` for you, but it probably won't look the way you expect. Your searches will probably vary, but I found Newtonsoft's JSON.NET, which handled `ExpandoObject` right out of the [NuGet box](http://nuget.org/List/Packages/Newtonsoft.Json). Then I stumbled on [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text) (also ["NuGettable"](http://nuget.org/List/Packages/ServiceStack.Text)). While it does even weirder things than the .NET serializer with `ExpandoObject`s, it supposedly does them very fast.

<!--more-->

### Test code

    dynamic test = new ExpandoObject();
    test.x = "xvalue";
    test.y = DateTime.Now;

### BCL JavaScriptSerializer (`using System.Web.Script.Serialization;`)

    JavaScriptSerializer javaScriptSerializer = new JavaScriptSerializer();
    string jsonOfTest = javaScriptSerializer.Serialize(test);
    // [{"Key":"x","Value":"xvalue"},{"Key":"y","Value":"\/Date(1314108923000)\/"}]

Not quite what I was looking for but it makes sense if you realize that `ExpandoObject` plays very nicely with `IDictionary<string, object>`. By using [some code borrowed from StackOverflow](http://stackoverflow.com/questions/5156664/how-to-flatten-an-expandoobject-returned-via-jsonresult-in-asp-net-mvc/5913094#5913094) (not the accepted answer, but I like it) and [theburningmonk.com](http://theburningmonk.com/2011/05/idictionarystring-object-to-expandoobject-extension-method/), you put together a custom serializer for `ExpandoObject` and you can get something more typical of what went into assembling the object.

    /// <summary>
    /// Allows JSON serialization of Expando objects into expected results (e.g., "x: 1, y: 2") instead of the default dictionary serialization.
    /// </summary>
    public class ExpandoJsonConverter : JavaScriptConverter {
        public override object Deserialize(IDictionary<string, object> dictionary, Type type, JavaScriptSerializer serializer) {
            // See source code link for this extension method at the bottom of this post (/Helpers/IDictionaryExtensions.cs)
            return dictionary.ToExpando();
        }
        public override IDictionary<string, object> Serialize(object obj, JavaScriptSerializer serializer) {
            var result = new Dictionary<string, object>();
            var dictionary = obj as IDictionary<string, object>;
            foreach (var item in dictionary)
                result.Add(item.Key, item.Value);
            return result;
        }
        public override IEnumerable<Type> SupportedTypes {
            get {
                return new ReadOnlyCollection<Type>(new Type[] { typeof(ExpandoObject) });
            }
        }
    }

    JavaScriptSerializer javaScriptSerializer = new JavaScriptSerializer();
    javaScriptSerializer.RegisterConverters(new JavaScriptConverter[] { new ExpandoJsonConverter() });
    jsonOfTest = javaScriptSerializer.Serialize(test);
    // {"x":"xvalue","y":"\/Date(1314108923000)\/"}

### Newtonsoft Json.NET

    string jsonOfTest = Newtonsoft.Json.JsonConvert.SerializeObject(test);
    // {"x":"xvalue","y":"\/Date(1314108923000-0600)\/"}

That worked exactly as I expected. If I can get JSON.NET to work consuming JSON under MonoTouch, it will make my life quite easy; more to come on that.

### ServiceStack.Text

    string jsonOfTest = JsonSerializer.SerializeToString(test);
    // ["[x, xvalue]","[y, 8/23/2011 08:15:23 AM]"]

ServiceStack's JSON serialization system does something similar to the .NET `JavaScriptSerializer`, but not quite the same. I haven't spent enough time with ServiceStack to know how this syntax will work out on consumption by another deserializing system, but I suspect this may be something that only ServiceStack would handle correctly.

Unfortunately, the author of the project was nice enough to confirm that [ServiceStack.Text does not currently afford the same extensibility as the .NET `JavaScriptSerializer`](http://stackoverflow.com/questions/7141767/how-to-serialize-expandoobject-using-servicestack-jsonserializer) for overriding its default behavior in this situation. He did welcome a [pull request](http://help.github.com/send-pull-requests/) which I will look into.

ServiceStack.Text also doesn't appear to support deserializing into an `ExpandoObject` as this resulted in an empty object.

    dynamic testDeserialization = ServiceStack.Text.JsonSerializer.DeserializeFromString<ExpandoObject>(jsonOfTest);

I haven't confirmed if ServiceStack.Text deserializing works under MonoTouch yet. If it does, it would be worthwhile to have it running API JSON generation as well as the client-side JSON consumption since there is [evidence it performs quite nicely](http://www.servicestack.net/mythz_blog/?p=344).

_UPDATE_

I slapped together a new MonoTouch project in MonoDevelop and tossed in ServiceStack.Text's DLLs with a few bits of code and confirmed it works great for a deserializing JSON into a pre-defined object.

    public class TestClass {
        public int x { get; set; }
        public string x { get; set; }
    }
    TestClass result = ServiceStack.Text.JsonSerializer.DeserializeFromString<TestClass>("{\"x\":3,\"y\":\"something\"}");
    Console.WriteLine("result: x={0}, y={1}", result.x, result.y);
    // result: 3, something

_UPDATE (2012-04-05)_

I missed a blog entry on the author's blog describing how to get Json.NET to output [`DateTime` in different ways](http://james.newtonking.com/archive/2009/02/20/good-date-times-with-json-net.aspx). For example, if you prefer the ISO-8601 output, you would be able to tell it to use the `IsoDateTimeConverter`. When I went to update the test project, the latest version of [Json.NET (4.5 Release 1) now defaults to ISO-8601](http://james.newtonking.com/archive/2012/03/20/json-net-4-5-release-1-iso-dates-async-metro-build.aspx). Since I don't want to risk break an existing API, I tweaked the `DateFormatHandling` to make the output match the old default. Examples are hard to find since this is such a new release, so I slapped one together and [submitted it to the Json.NET docs](https://github.com/JamesNK/Newtonsoft.Json/pull/11) (now on GitHub).

    Newtonsoft.Json.JsonSerializerSettings settingsWithMicrosoftDateFormat = new Newtonsoft.Json.JsonSerializerSettings() { DateFormatHandling = Newtonsoft.Json.DateFormatHandling.MicrosoftDateFormat };
    string jsonOfTest = Newtonsoft.Json.JsonConvert.SerializeObject(test, settingsWithMicrosoftDateFormat);
    // {"x":"xvalue","y":"1333640263042-0600"}

Unfortunately, even Json.NET 4.5 is appending a timezone offset to the `DateTime` serialization that isn't found in the .NET implementation. I'll look into a custom implementation of `DateTimeConverterBase` and I have posted this as a [question on StackOverflow](http://stackoverflow.com/questions/10033612/prevent-json-net-4-5-from-appending-timezone-offset-when-using-microsoftdateform).

### Additional Notes

I haven't played much with what problems may arise with the various representations of DateTime objects on the consumption side, but they definitely all handled it differently here.

### Source Code

To get all the code in an ASP.NET MVC project (download, load solution, hit F5 [I hope]), check out [the bitbucket.org repository for this post](https://bitbucket.org/patridge/expandoobject-json-serialization-tests).

---

## Comments

* **Charles Egan**, _2012-06-14 17:48:14 +0000_

    > This was exactly what I was looking for.  Thanks so much!

* **Jon Canning**, _2012-07-10 08:32:51 +0000_

    > Hi Adam,  I threw this together since I wanted to quickly test some APIs with dynamics and would appreciate your feedback:
    >
    > http://nuget.org/packages/cerealbox
    > https://github.com/JonCanning/CerealBox

* **Will**, _2013-11-07 05:35:46 +0000_

    > Try this:
    >
    > ```
    > public override IDictionary Serialize(object obj, JavaScriptSerializer serializer)
    > {
    >   return (obj as IDictionary).ToDictionary(kvp =&gt; kvp.Key, kvp =&gt; kvp.Value);
    > }
    > ```
