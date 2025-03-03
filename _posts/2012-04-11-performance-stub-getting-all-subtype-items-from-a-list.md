---
layout: post
title: "Performance Stub: getting all subtype items from a list"
author: "Adam Patridge"
date: Wed, 11 Apr 2012 13:51:28 +0000
tags: c-sharp performance performance-stub
category: dev
excerpt_separator: <!--more-->
---

## Performance Stubs

These blog posts are simply times I wanted to identify the fastest method for accomplishing a particular goal. As I write the code, I like to make some light notes of alternatives while driving forward with the first implementation that makes it from my brain to my fingers.

When I get the chance, I can go back and flesh out the two versions and drop them into some basic `Stopwatch` timing to determine which is better in terms of raw speed. Factoring those results with clarity of code, I have a method I will likely choose the next time I need the same feature.

## Goal

Given a particular `IEnumerable`, find all the elements that are of a particular type (subclass, in this case).

<!--more-->

### Method 1: Select As, Where Not Null

The [`as` operator](http://msdn.microsoft.com/en-us/library/cscsdfbt(v=vs.71).aspx) with convert between two types with one nice tweak. If the cast cannot happen, it results in `null`.

In this version, we massage the list by hitting all items with an `as` cast and then filter out the ones that didn't cast successfully.

    public static IEnumerable<SubSomething> SelectAsWhereNotNull(IEnumerable<Something> source) {
        return source.Select(item => item as SubSomething).Where(item => item != null);
    }

### Method 2: Where Is, Cast

There is also the [`is` operator](http://msdn.microsoft.com/en-us/library/scekt9xw(v=vs.71).aspx). Instead of returning a cast result, it simply returns a boolean for whether the instance can be cast as a given type. From there, we use another `Linq` extension method: [`Cast<TResult>`](http://msdn.microsoft.com/en-us/library/bb341406.aspx).

In this version, we filter the original list to just those that can make the cast successfully, then `Cast` those items to the final desired type.

    public static IEnumerable<SubSomething> WhereIsCast(IEnumerable<Something> source) {
        return source.Where(item => item is SubSomething).Cast<SubSomething>();
    }

## Test

I built up a list (n=1,000,000) of parent type `Something` but make every other item a `SubSomething` (which is a subclass of `Something`). Then, I run that list through both methods 1000 iterations each.

## Results

### Clarity

Neither method is particularly difficult to decipher. The `Select`/`Where` method takes a few more characters, but not enough to hurt.

### Speed

It takes quite a list for these tests to even register with average [ticks](http://msdn.microsoft.com/en-us/library/system.datetime.ticks.aspx). That said, there is little gain to be had here over small lists. Once list lengths are sufficiently large, method 1 took the lead.

<table>
  <caption>For n=1,000,000 and 1000 iterations (run on 2.4GHz AMD Phenom 9750 with 8GB RAM)</caption>
  <tr>
    <th>Average</th>
    <th>Method</th>
  </tr>
  <tr>
    <td>1.0 ticks</td>
    <td><code>{IEnumerable&lt;Something&gt;}.Select(item =&gt; item as SubSomething).Where(item => item != null)</code></td>
  </tr>
  <tr>
    <td>1.7 ticks</td>
    <td><code>{IEnumerable&lt;Something&gt;}.Where(item => item is SubSomething).Cast&lt;SubSomething&gt;()</code></td>
  </tr>
</table>

## Performance Testing Framework

The original source for the test framework code evolved from a [StackOverflow answer about converting byte arrays to hexadecimal strings](http://stackoverflow.com/questions/311165/how-do-you-convert-byte-array-to-hexadecimal-string-and-vice-versa-in-c/624379#624379). --That code is available in a [bitbucket repo](https://bitbucket.org/patridge/convertbytearraytohexstringperftest). I have adapted it here for these two methods.-- That code is available in the [framework repo](https://github.com/patridge/PerformanceStubs).

Rather than post another repo for each test, I am going to abstract out a testing framework for these posts. That repo will show up soon and I will update this post accordingly.

### Update 2013-01-22

I finally refined the testing framework code I was using. While I won't pretend this code couldn't use some improvements, feel free to hack away with it. The code is [available on GitHub](https://github.com/patridge/PerformanceStubs).

To add a new case to an existing test:

1. Add the new static method (`Func<byte[], string>`) to /Tests/ConvertByteArrayToHexString/Test.cs.
1. Add that method's name to the `TestCandidates` return value in that same class.
1. Make sure you are running the input version you want, sentence or text, by toggling the comments in `GenerateTestInput` in that same class.
1. Hit <kbd>F5</kbd> and wait for the output (an HTML dump is also generated in the /bin folder).

To add a new test fixture entirely, just write up something that implements IPerformanceTest in that project's namespace.
