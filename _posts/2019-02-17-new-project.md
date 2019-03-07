---
layout: post
title:  "Bluetooth Build Button, Part 1: project planning"
date:   2019-02-26 21:42:00
author: garyjohnson
category: articles
published: true
image:
  feature: button01/button-header.jpg
---

It's time for a new project! I've been starting projects and enjoying not finishing them, but I've been feeling the itch to carry something small and focused all the way through design, documentation, and delivery.

## The Idea 

A few years ago I built a build [button](https://www.automationdirect.com/adc/shopping/catalog/pushbuttons_-z-_switches_-z-_indicators/22mm_metal/mushroom_pushbuttons_illuminated_-a-_non-illuminated/gcx1137) to run my tests -- a simple [Teensy](https://www.pjrc.com/teensy/teensy31.html) that connects as a HID keyboard and sends a key combo when the button is pushed. In IDEs, I bound the key combo to running tests, and in iTerm to send `rake spec \n` (not very flexible for moving between types of projects).

![original button](../../images/button01/button.jpg)

Moving between computers with USB and USB-C ports makes the original button a little more annoying to use, so I'd like to go wireless. In 2019 there's lots of great bluetooth hardware available. 
The [Adafruit Feather nrf52832](https://www.adafruit.com/product/3406) is relatively inexpensive, has a battery charging circuit, [can emulate a bluetooth keyboard](https://learn.adafruit.com/bluefruit-nrf52-feather-learning-guide/blehidadafruit), and I happen to have two of them already.

![feather nrf52832](../../images/button01/nrf.jpg)

So that's the plan -- a bluetooth button that can run my tests and has an internal battery with USB charging.

![button sketch](../../images/button01/button-sketch.jpg)

## Scope Creep

There's a couple other things I'd like to tackle as part of this project, all of them negotiable:

* Utilizing [deep sleep capabilities on the nrf52](https://github.com/arduino-libraries/ArduinoLowPower/blob/master/examples/PrimoDeepSleep/PrimoDeepSleep.ino) for really, really long battery life
* Running the correct test command based on the project or directory
* Ability to easily open and close the enclosure without additional hardware:
![snap design](../../images/button01/button-snap.jpg)
* I want the button to provide feedback at minimum, and maybe have the capability of being an I/O device. Could be some combination of sound, vibration, and light. Here's some ideas incorporating an [Adafruit NeoPixel Ring](https://www.adafruit.com/product/1586):
![io](../../images/button01/button-io.jpg)


## Next Steps

That's it! Next post I'll dive into building the proof-of-concept:

* Press the built-in button on the nrf52 and be able to trigger a bluetooth keyboard press
* Wiring up the button, nrf52, battery, power switch

[Bluetooth Build Button, Part 2: the proof-of-concept](/articles/button-mvp)
