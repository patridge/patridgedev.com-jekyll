---
layout: post
title: "Add variables to your custom `dotnet new` template"
date: Mon, 22 Oct 2018 20:00:49 +0000
tags: dotnet template
# category: dev
excerpt_separator: <!--more-->
---

Previously, I covered [creating your first custom `dotnet new` template](https://www.patridgedev.com/2018/10/09/making-a-custom-dotnet-new-template/). Now, let's work on customizing the content our template generates based on inputs provided via the command line. I’ll be working from the [same custom template from that post](https://www.patridgedev.com/2018/10/09/making-a-custom-dotnet-new-template/), which is just a `dotnet new console` output with its own **.template.config** setup. If you want a starter template project to get you going, use the template from the [**1-custom-template** folder](https://github.com/patridge/demo-custom-dotnet-template/tree/master/1-custom-template) from the [Git repo from that blog post](https://github.com/patridge/demo-custom-dotnet-template).

> This is the second in a collection of posts about [creating custom templates for the `dotnet new` system](https://www.patridgedev.com/tag/template/).

<!--more-->

## Your first input parameter

When you are generating content from a template, your first parameter for controlling things is built in to the SDK. There are two variables used to decide where to put the generated content: `--output` and `--name`. If you provide a path via `--output "some/path"`, the content will be generated in that location. If, in your template's **template.json** file, you set `"preferNameDirectory": true`, you can set the same path via `--name "some/path"`. (If you have the `preferNameDirectory` variable set to true and provide both an `output` and `name` variable, it will use the `output` value.) Without either command-line parameter, the new content will be generated in the current directory.

Passing in more useful data for your template requires some additional changes to the **template.json** configuration.

## Taking inputs for generating content

To start consuming inputs to alter your generated content, add a new object to the template configuration JSON in **template.json**.

<!-- language: javascript -->

    {
        "$schema": "http://json.schemastore.org/template",
        ...
        "symbols": {
            "helloMessage": {
                "type": "parameter",
                "replaces": "Hello from a new template!"
            }
        }
    }

This sets up your template to take a `helloMessage` parameter via the command line like this.

<!-- language: bash -->

    dotnet new console-awesome --helloMessage "Hello from a command line override!"

The resulting Program.cs file will have a call using our custom parameter.

<!-- language:csharp -->

    Console.WriteLine("Hello from a command line override!");

If we don't pass in that `helloMessage` parameter, the substitution is made with the value set in  a symbol's `defaultValue` field.

When the content is generated, it will process the template files and replace all instances of “Hello from a new template!“ with the provided parameter value. We only have one instance here, but you could have several if your template requires it.

## Making the output more predictable

Since the template system does a find-and-replace on our symbols defining a `replaces` value, I want to make sure it doesn't accidentally find any other instances I didn't intend for it to replace. To do that, I will typically use a placeholder string that is always replaced, like a template token.

Most template examples I have looked over seem to put default data in the file to be processed and replace that value with the `replaces` value. I tend to use a placeholder string and a `defaultValue` within the symbol instead. They both result in the same generated content, so let your own preferences guide you here. I feel like I’m less likely to have identical text elsewhere that will get accidentally replaced as a "false positive" when I use replacement tokens than with actual content.

In the previous example, I would replace the `WriteLine` parameter in C# with something like `Console.WriteLine(”{helloMessage}”)` and configure the symbol like this.

<!-- language: json -->

    {
        "$schema": "http://json.schemastore.org/template",
        ...
        "symbols": {
            "helloMessage": {
                "type": "parameter",
                "replaces": "{helloMessage}",
                "defaultValue": "Hello from a new template!"
            }
        }
    }

You will end up creating a symbol for each input you want to accept from the command line, and several more advanced steps. You will also create symbols for some of the processing you wish to do to other inputs, for example changing the letter casing of a provided value.

## Changing input parameter casing

There are lots of things we can do to incoming parameter values, but we’ll start with a simple one: forcing something to all uppercase or lowercase. If you're taking in an input from a user of your template, you can’t always rely on them providing the expected input. For example, if you expect an ID field in your template to have letters in uppercase only, you will want to set it up to enforce that when the templated content is generated.

We add this string modification as a second symbol that will derive a modified value from the original, keeping the original also available if you need it. To do so, return to your **.template.config** > **template.config** file.

In the **symbols** section of the config file, add a new symbol to capitalize the existing `helloMessage` symbol; I don't know if there's a best practice, but I try to name my modifier symbols after what they do. So, in this case, name your new symbol `helloMessageUpper`. Let's see what that will look like.

<!-- language: json -->

    "symbols": {
       "helloMessage": {
           "description": "",
           "type": "parameter",
           "replaces": "{helloMessage}",
           "defaultValue": "Hello from a new template!"
       },
       "helloMessageUpper": {
           "type": "generated",
           "generator": "casing",
           "parameters": {
             "source": "helloMessage",
             "toLower": false
           },
           "replaces": "{helloMessageUpper}"
       }
   }

First, we still have our input symbol, `helloMessage`. Then, we add our new `helloMessageUpper` symbol. To tell the system we are making a computed symbol, we set the `type` of our symbol to `generated. We then need to tell it what kind of `generator` to use: `casing` in this case, to tell the system we are going to modify the casing on some other symbol.

When you use a generator symbol, you often require some additional configuration, which is done with a `parameters` configuration object. In this case, you provide a `source` symbol, the input to our casing generator, and the direction to adjust the letter casing in a `toLower` boolean value.

The `toLower` boolean value doesn't have any sort of `toUpper` equivalent. As you might expect, if you set it to `true`, it will result in a lowercase symbol you can use. What may not be as obvious, though, is that setting `toLower` to `false` will result in an uppercase symbol.

With the generator configured, we simply add a `replaces` value again to tell it what gets our new uppercase-transformed input in the template output.

To test our output, let's add another line in the template C# source.

<!-- language:csharp -->

    Console.WriteLine("{helloMessage}");
    Console.WriteLine("{helloMessageUpper}");

Now, we need to install or modified template and generate content from it. From your template directory, make sure you have the latest installed.

<!-- language:console -->

    dotnet new --install .

If we generate a project from our modified template without any command-line parameters, we get the default symbol value. That default is also fed into the generated symbol for it's upper-case equivalent.

This command…

<!-- language: bash -->

    dotnet new console-awesome --output TestRunNoParameter

…results in a **TestRunNoParameter/Program.cs** containing these calls. (Remember, the `output` parameter will generate the content in a folder with that name, used here to keep our test output organized.)

<!-- language:csharp -->

    Console.WriteLine("Hello from a new template!");
    Console.WriteLine("HELLO FROM A NEW TEMPLATE!");

And when we provide our own command-line parameter, it substitutes the provided value as above and also passes that value into the upper-case generator.

This command…

<!-- language: bash -->

    dotnet new console-awesome --helloMessage "Hello from a command line override!" --output "TestRunParameterOverride"

…results in these calls in Program.cs.

<!-- language:csharp -->

    Console.WriteLine("Hello from a command line override!");
    Console.WriteLine("HELLO FROM A COMMAND LINE OVERRIDE!");

## Check your work

You can now start introducing variables into your `dotnet new` templates. If you want to compare your work to a template created by following along, check out the [**2-input-parameters** folder](https://github.com/patridge/demo-custom-dotnet-template/tree/master/2-input-parameters) of the [GitHub repo created for this blog post series](https://github.com/patridge/demo-custom-dotnet-template).

There's so much more that can allow you to do even more advanced things with your templates, some of which I hope to cover in more posts. If there’s something cool that I need to cover, though, [reach out to me on Twitter: @patridgedev](https://twitter.com/patridgedev).
