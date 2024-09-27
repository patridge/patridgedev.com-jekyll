---
layout: post
title: "Return of Netduino, .NET on small hardware"
author: "Adam Patridge"
date: Mon, 11 Sep 2017 14:00:56 +0000
tags: Netduino C#
category: dev
excerpt_separator: <!--more-->
---

A little piece of magic wandered into my Twitter feed recently. Between the Windows IoT stuff I messed with [this](https://twitter.com/PatridgeDev/status/843113990519771140) [summer](https://twitter.com/PatridgeDev/status/844553311059693569), controlling LEDs via Xamarin.Forms, and this awesome news, I'm bound to be learning the hardware side of things a little better.

<blockquote class="twitter-tweet"><p lang="nl" dir="ltr">Netduino is back! <a href="https://t.co/gU2CNIWFYE">https://t.co/gU2CNIWFYE</a></p>&mdash; Netduino (@netduino) <a href="https://twitter.com/netduino/status/888092844337971200">July 20, 2017</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<!--more-->

That's right, Netduino has returned; this time [Wilderness Labs](https://www.wildernesslabs.co/) has taken the reins (with [Bryan Costanich](https://twitter.com/bryancostanich) heading things up). Netduino is a hardware platform, similar to Arduino with lots of input and output options through a bunch of pins on the board, that allows interacting with it using .NET. Not only has Wilderness Labs resurrected Netduino, they have released three new boards: [Netduino 3](http://amzn.to/2wdBl0L), [Netduino 3 Ethernet](http://amzn.to/2xlJm3E), and [Netduino 3 WiFi](http://amzn.to/2vfFmVb).

According to the [Wilderness Labs Netduino docs](http://developer.wildernesslabs.co/Netduino/About/), these new boards are as fast as ever, all running Cortex-M4 processors at 168MHz, and with more flash storage and a bump in RAM: the base Netduino 3 has 384KB of flash (matching the old Netduino 2 Plus), and the Netduino 3 Ethernet and Netduino 3 WiFi both have a bump to 1,408KB. If it wasn't obvious, the Netduino 3 Ethernet board also offers a 10/100 Ethernet port, and the Netduino 3 WiFi has an 802.11b/g/n wi-fi adapter. So, they are as fast as ever and with more resources to do what you tell them.

I have been interested in Netduino since I first heard about it, though I didn't buy one to play with until the release of the prior Netduino 2 Plus. All I had ever done with that system was blink the on-board LED. A few hours after I read that tweet, I was digging through my box-o'-electronics until I found it. This time, I decided I would do something more useful, or at least more fun.

## Getting hardware rolling [again]

Since I wasn't entirely sure what firmware was installed on my Netduino, I decided to [install the latest from Wilderness Labs](http://developer.wildernesslabs.co/Netduino/About/Updating_Firmware/). I figured there was no sense in finding a bug that was fixed in the later firmware versions.

Unfortunately, every time I tried to flash the firmware on my MacBook, it would get stuck. After several unsuccessful attempts to debug the issue, I randomly ended up using a different USB cable. Everything flashed perfectly after that, and I tossed the defective USB cable. While it may not help with my faulty cable issue, [Alex Corrado](https://twitter.com/chknofthescene) has since [updated the error handling on the macOS firmware update tool](https://github.com/WildernessLabs/Netduino-Updater/pull/3), so you may have not have as much trouble debugging issues by the time you read this.

## Getting an IDE rolling

What little code I wrote the last time I did Netduino development was probably on Visual Studio 2013. While the Netduino Discord channel looks like Netduino is quickly moving to Visual Studio 2017 and Visual Studio for Mac, it's not quite 100 percent there yet. So, I busted out my older MacBook with Xamarin Studio. (I could have also installed Visual Studio 2015, but having spent so much time with 2017's more self-contained installs makes me not want to return to the previous era.)

Since I was going the Xamarin Studio route, I added the [**Micro Framework** add-in/extension as instructed](http://developer.wildernesslabs.co/Netduino/Getting_Started/). A quick restart of the IDE, and I was up and rolling, blinking the same LED I'd blinked a few years back.

## What's next? 

I have played with a few sensors and LEDs already, and some more code-happy posts will be coming soon.

So far, I have a few random toys to play with: a water-resistant temperature sensor, a [BME280 temperature/pressure/altitude sensor](http://amzn.to/2viNF2v), and an ultrasonic range sensor

Even if the Netduino is overkill, I might start working on a small temperature-monitoring system feeding to Azure IoT Hub with an app front-end for our chicken coop. Or maybe a silly well-sensored aquaponics system. As much fun as blinking LEDs can be, what other semi-useful things should I be trying to make? Drop me a [line on Twitter](https://twitter.com/patridgedev) if you have any suggestions.
