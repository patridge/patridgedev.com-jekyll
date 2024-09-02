---
layout: post
title: "Fallback variables in `dotnet new` templates"
date: Mon, 29 Oct 2018 14:00:25 +0000
tags: dotnet template
# category: dev
excerpt_separator: <!--more-->
---

Previously, we created our first custom `dotnet new` template and added our first input parameter for it. Next, let's get a little more advanced with our parameters to make our life easier. In this post, we'll create a template symbol that will function like a coalesce operator, taking a preferred input but falling back to a different input if the preferred value is not found.

> This is the third in a collection of posts about [creating custom templates for the `dotnet new` system](https://www.patridgedev.com/tag/template/).

<!--more-->

## Sample code

This is the third in a series of posts about [creating custom templates for the `dotnet new` system](https://www.patridgedev.com/tag/template/). I’ll be working from the results of the prior blog post, but the approach can be used to add this functionality to any `dotnet new` template. If you want a starter template project to get you going, you can clone the [Git repo from that blog post](https://github.com/patridge/demo-custom-dotnet-template) and start from the prior **2-input-parameters** template.

## Why a coalesce/fallback symbol?

There are bound to be several great reasons to need a fallback variable, or one that coalesces between multiple potential inputs, but here are a few I've run across so far.

### Provide a generated value with a manual override

If you remember from the case-changing symbol from the prior post, there are symbols that can be computed in some fashion. In that case, we were adjusting the letter casing of another input to be upper-case, but there are several other [`type: generated` symbol options](https://github.com/dotnet/templating/wiki/Available-Parameter-Generators). If you have a default of generating a template value but want to allow the user to submit an override value, the coalesce generator is perfect.

I've used this previously when [templating a repo license file to generate the copyright year value but allow for a user-submitted override for existing](https://github.com/patridge/dotnet-new-project-helper/blob/5726f7b4c0b4bc501a628980f5ca8e52c3363659/add-license/.template.config/template.json#L10-L28), older projects. (Yes, that is a NuGet package with `dotnet new` templates for creating more `dotnet new` templates.)

### Provide a fallback to another user-provided symbol

Alternatively, if you have a template value that might need to be set manually, but should adopt a value from another symbol when the manual value isn't present. This is also easily accomplished with a `generator: coalesce` symbol by pointing it to the manual input value for the primary symbol and the back-up input value for the fallback symbol.

On a recent project involving a content management system, pages had several YAML values. One value for content would be the full-length title to display at the top of the content page, but there was also a shortened title available for lengthier titles to display in width-constrained locations. If you didn't need a special shortened variant of the title, you could omit that value and it would simply use the full-length title for both YAML values.

## Create a coalesce symbol

At its core, a coalesce symbol has two parameter values: `sourceVariableName` and `fallbackVariableName`. These two values can point to any other symbols you have defined, whether they are other input parameters or generated values or a mix of the two.

<!-- language:json -->

       "copyrightYear": {
           "type": "parameter"
       },
       "copyrightYearGenerated": {
           "type": "generated",
           "generator": "now",
           "parameters": {
               "format": "yyyy"
           }
       },
       "copyrightYearReplacer": {
           "type": "generated",
           "generator": "coalesce",
           "parameters": {
             "sourceVariableName": "copyrightYear",
             "fallbackVariableName": "copyrightYearGenerated"
           },
           "replaces": "{copyrightYear}"
       },

In this example from the repo license template example mentioned previously, we have a `copyrightYear` parameter optionally provided by the person generating content. Then, we have a generated four-digit year symbol (e.g., "2018"). Last, there is a coalesce symbol that points its `sourceVariableName` to the provided parameter but uses the generated value via the `fallbackVariableName` when a value isn't provided. The result of this source-plus-fallback coalesce is then used to replace any instances of `{copyrightYear}` in the template content.

## Check your work

If you got lost along the way or need something to compare to, look in the [**3-coalesce-parameters** folder](https://github.com/patridge/demo-custom-dotnet-template/tree/master/3-coalesce-parameters) of the [sample Git repo with the resulting templates](https://github.com/patridge/demo-custom-dotnet-template) from following along with this blog post series.

You can now start introducing coalesce, or fallback, variables into your `dotnet new` templates. As always, there's so much more that can allow you to do even more advanced things with your templates, some of which I hope to cover in more posts. If there’s something cool that I need to cover, though, [reach out to me on Twitter: @patridgedev](https://twitter.com/patridgedev).
