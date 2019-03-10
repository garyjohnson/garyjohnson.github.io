---
layout: post
title:  "Bluetooth Build Button, Part 3: the enclosure rough-ins"
date:   2019-03-04 10:36:00
author: garyjohnson
category: articles
published: true
image:
  feature: button03/header.png
---

With the [core idea now functional](/articles/button-mvp), it's time to start working on a proper enclosure.

For building models, I've been using [Autodesk Fusion 360](https://www.autodesk.com/products/fusion-360/overview). It's a powerful parametric modeling tool and has [free non-commercial licensing](https://www.autodesk.com/campaigns/fusion-360-for-hobbyists). The license is only one year long, but can be renewed each year.

## What is parametric modeling?

Let's compare **direct** and **parametric** modeling software for a minute, because the distinction is important. 

A couple years back before I knew the difference, I was working in a **direct modeling** tool, the now-defunct [Autodesk 123D Design](https://www.autodesk.com/solutions/123d-apps) (which was a wonderful, simple, intuitive tool, but that's neither here nor there). I ended up designing the most complex model I've built yet -- an internet-connected, [Raspberry Pi](https://www.raspberrypi.org)-powered, touchscreen RGB lamp for an [Introduction to Connected Devices course at Case Western Reserve University](http://bulletin.case.edu/course-descriptions/eecs/). 

It was pretty complex -- it was tooless and had an interlocking sliding-rail system for opening and closing, as well as mounting features for the hardware inside:

![lampi](../../images/button03/lampi.gif)

With **direct modeling**, you've got the model you're working on and ... that's it. If you realize you need to change something simple, shave a millimeter here or there, you'll probably be fine. If you need to go back and change something fundamental, something that _other_ features of the model are built upon, you'll need to go back to an older version of the file (you _did_ put the file in version control, right?), make the change, and then _manually redo_ any work that came afterwards -- this could be hours of work, and the chance of error is pretty huge.

As you can imagine, this became _quickly_ untenable for changes of any appreciable size. It was pretty humbling to realize how quickly my ignorance of tooling choice turned into something resembling a heap of tech debt.

Direct modeling was a simple, useful tool being abused with a mismatched use case. If you're modeling anything (mechanical) of complexity you may _ever_ want to change, **parametric modeling** is the way to go. 

With parametric modeling, your source-of-truth is not the model in front of you -- it's the history of changes you made to get there, and your current model is a byproduct of those changes. Fusion 360 allows you to easily traverse that history:

![parametric history](../../images/button03/parametric-history.gif)

And because of that change in perspective, making fundamental modifications becomes incredibly easy. Think of it like a `git rebase` -- you're inserting a change earlier in history, and then replaying the rest of the changes on top. When changes aren't compatible, you even have the opportunity to resolve conflicts:

![parametric modifications](../../images/button03/parametric-modification.gif)

So, [parametric modeling is really powerful](https://www.youtube.com/watch?v=n-IsRIFzlHs) and it's going to allow me to quickly iterate on this enclosure moving forwards.

## Gathering parts

When modeling an enclosure, the first thing I do is build rough-ins -- replicas of all the parts I'm going to put in the enclosure. They don't need to be perfect, but should have accurate measurements for any mounting holes, ports, anything where alignment is key. 

We've got a couple parts to rough-in. The button and contact switch assembly:

![button](../../images/button03/button.jpg)

One of these batteries -- we haven't figured out how much juice (or room) we need yet:

![battery](../../images/button03/battery.jpg)

A power switch:

![switch](../../images/button03/switch.jpg)

And the Feather:

![feather nrf](../../images/button03/nrf.jpg)

We've got to measure all this stuff to actually model it. I recommend using [digital calipers](https://www.generaltools.com/6-in-steel-digital-caliper-1) for this work:

![measure](../../images/button03/measure.jpg)

## Modeling the rough-ins

Let's start with the switch. 

When doing parametric modeling in Fusion, you'll be working a lot with sketches and then extruding them into 3D shapes. Sketches are great because they allow you to define complex layout rules in a 2D space that can react automatically to future modifications.

For the switch, we'll start with a rectangle sketch the width and depth of the switch, and then extrude the height:

![switch extrude](../../images/button03/02-extrude.png)

Then we'll make another sketch on the bottom of our new switch and build a rectangle for each pin:

![switch pin sketch](../../images/button03/04-pins-sketch.png)

Extrude those pins, extrude a button, you get the idea. After a few more rectangles and a few more extrusions, we have something resembling a switch.

![switch rough in](../../images/button03/05-switch.png)

The top of the switch is longer because we're roughing in the _full range of motion_ of the switch, ensuring when we test for placement, the switch will be able to move back and forth freely.

![switch model](../../images/button03/switch-model.jpg)

Okay, so I went a little lazy on the battery. [LiPo batteries have been known to expand](https://www.reddit.com/r/techsupportgore/comments/2g78tf/extreme_battery_expansion_on_an_iphone/), so I was a little generous with the dimensions, and made a little nub to indicate where the wires exit.

![battery model](../../images/button03/battery-model.jpg)

The [Adafruit Feather](https://www.adafruit.com/feather) line has standardized ports and mounting holes, so I assumed I would be able to find an existing model of _any_ Feather, and [sure enough, I did](https://gallery.autodesk.com/fusion360/projects/adafruit-wiced-feather):

![feather model](../../images/button03/feather-model.png)

Finally, the button, which I took extra care with, since it has such a big impact on the available space in the enclosure.

![button model](../../images/button03/contact-block-lofted.png)

## Modeling the enclosure

I'd like the enclosure to roughly match the diameter of the button, so let's just drop this stuff in a cylinder and see how it fares:

![enclosure model](../../images/button03/enclosure-rotate.gif)

That battery looks like a problem. Additionally, I'd like the top of the enclosure to twist off, which means the button has to be free to twist with it:

![enclosure spin model](../../images/button03/enclosure-button-spin.gif)

Okay, so we'll go model the smaller battery and start there. There's another issue, which is that the USB port and power switch don't really work with a rounded surface:

![port flat](../../images/button03/port-flat.gif)

Looks like we're going to need to [build a flat into our cyclinder](https://www.apple.com/mac-pro/).

## Scope Creep

Finally, the contact switch on the button really takes up a ton of good space -- this is something I'd like to address, but it works and building an alternative is going to distract us from delivering V1. Let's put it on our nice-to-haves list for now.

## What's next 

With the rough-ins done, we managed to visualize some potential issues. We'll move to the smaller battery and then get to work on [making a v1 enclosure](/articles/build-button-v1) we can 3D print and test out.
