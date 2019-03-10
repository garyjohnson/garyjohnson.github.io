---
layout: post
title:  "Bluetooth Build Button, Part 4: v1.0 complete"
date:   2019-03-09 19:57:00
author: garyjohnson
category: articles
published: true
image:
  feature: button04/header.png
---

With the [rough-ins in place](/articles/enclosure-rough-ins), we can build out the enclosure.

## Battery

After seeing that the 2000mAh battery wasn't going to fit, I moved to modeling the 500mAh one and tucked it under the Feather:

![battery](../../images/button04/battery.png)

Eventually I'd like everything to snap in without screws or hardware, but snaps take some modeling work and iteration -- screw holes take about three seconds, so in they go.

## Flat for ports

We need a flat surface for our USB port and switch, so once the position of the Feather is determined, we can build a sketch and cut out a flat.

![flat](../../images/button04/flat.png)

![flat](../../images/button04/flat2.png)

And with that, we have the base of our enclosure:

![enclosure](../../images/button04/enclosure.gif)

## Lid

The lid is really just a couple of concentric circles extruded to different lengths, and takes about a minute to rough out.

![lid](../../images/button04/lid.png)

Before we print, a quick fit check with all of our parts and rough-ins:

![assembled](../../images/button04/assembled.png)

Let's print!

## Printing

I picked up some silver Hatchbox PLA for button printing. The silver should look nice with the red button.

![hatchbox pla](../../images/button04/silver-pla.jpg)

An hour or two later and we've got a button!

<video controls="controls" name="print" src="../../images/button04/print.mp4" poster="../../images/button04/preview.png" preload="auto"></video>
<br>

Mounting the switch will take a little bit more work, and there's plenty learn without it, so for now I've just wired it internally.

Because there are loose elements that may cause shorts, I've covered up the Feather with kapton tape as an insulator.

![switch](../../images/button04/switch.jpg)

Looks like the micro USB port lines up pretty good!

![usb lineup](../../images/button04/usb-lineup.jpg)

Here's our v1 build button. The print quality isn't great, the lid fits way too tight, but things look pretty okay when it's all assembled!

![final button](../../images/button04/final-button.jpg)

![final button back](../../images/button04/final-button-back.jpg)

Still feels pretty cool to be able to do this wirelessly:

<video controls="controls" name="v1" src="../../images/button04/v1.mp4" poster="../../images/button04/v1-preview.png" preload="auto"></video>
<br>

## Whats next

We've finished implementing the core idea, but there's a lot more finishing work that can go into this.

Here are the things that bug me, in the order of most annoying:

* Can't easily run the proper test suite in the command line.
* Having to open the case to switch power.

So next we'll start tackling those issues, and then we can move onto enhancing the button.
