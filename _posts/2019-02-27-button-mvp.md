---
layout: post
title:  "Bluetooth Build Button, Part 2: the mvp"
date:   2019-02-27 7:59:00
author: garyjohnson
category: articles
published: true
image:
  feature: button02/header.jpg
---

Well it didn't take much to get the [core idea](/articles/new-project) working.
<br>
<video class="video" controls="controls" name="mvp" src="../../images/button02/button-mvp.mp4"></video>
<br>
What you're seeing here is the [Adafruit Feather nrf52832](https://www.adafruit.com/product/3406) acting as a bluetooth keyboard, paired to my Macbook, and sending `CTRL+ALT+F12` when I push the giant red button (mostly because it's unlikely to be used by _anything_). It's powered by a 500mAh lipo battery and not wired to the Macbook at all.

In the video, I have iTerm bound to run `rake spec \n` on `CTRL+ALT+F12`:

![iterm binding](../../images/button02/iterm-binding.png)

Functional, but only useful for Ruby projects. We'll consider [making this dynamic based on the current project directory](/articles/new-project/#scope-creep) a "stretch goal".

Adafruit provides a [library with examples](https://github.com/adafruit/Adafruit_nRF52_Arduino/blob/master/libraries/Bluefruit52Lib/examples/Peripheral/hid_keyboard/hid_keyboard.ino) for using the nrf52 as a bluetooth keyboard, which provided enough juice to get my own version up and running: 

[build-button.ino (GitHub)](https://github.com/garyjohnson/build-button/blob/cf1c7cafbe215add65a0cb004e41e90da2040dd3/firmware/build-button/build-button.ino)

## Code Example

We initialize Bluefruit BLEDis, and BLEHidAdafruit:

~~~c
// Radio level configuration
Bluefruit.begin();
Bluefruit.setTxPower(4);
Bluefruit.setName("Build Button");

// BLE device information service
bledis.setManufacturer("Gary Johnson");
bledis.setModel("Build Button");
bledis.begin();

// BLE HID service
blehid.begin();
~~~

And then start advertising as a HID keyboard:

~~~c
// start advertising our HID service
Bluefruit.Advertising.addFlags(BLE_GAP_ADV_FLAGS_LE_ONLY_GENERAL_DISC_MODE);
Bluefruit.Advertising.addTxPower();
Bluefruit.Advertising.addAppearance(BLE_APPEARANCE_HID_KEYBOARD);
Bluefruit.Advertising.addService(blehid);
Bluefruit.Advertising.addName();
Bluefruit.Advertising.restartOnDisconnect(true);
Bluefruit.Advertising.setInterval(32, 244);    // in unit of 0.625 ms
Bluefruit.Advertising.setFastTimeout(30);      // number of seconds in fast mode
Bluefruit.Advertising.start(0);                // 0 = Don't stop advertising after n seconds
~~~

When the button is pushed, we send our `CTRL+ALT+F12`:

~~~c
if ( digitalRead(PIN_BUTTON) == 1 ) {
  blehid.keyboardReport(KEYBOARD_MODIFIER_LEFTCTRL|KEYBOARD_MODIFIER_LEFTALT, HID_KEY_F12);
}
~~~

And send the key up event when done:

~~~c
blehid.keyRelease();
~~~

And that's it! We have a bluetooth keyboard -- check the [GitHub page for the complete implementation](https://github.com/garyjohnson/build-button/blob/cf1c7cafbe215add65a0cb004e41e90da2040dd3/firmware/build-button/build-button.ino).

## Scope Creep

I really wanted to avoid the Arduino IDE as much as possible -- I do a lot of my work in neovim from the command line. Building command line tooling also means that my build is automatable on a headless system, which I always see as a virtue when working remotely through SSH or deploying using a [CI / CD pipeline](https://circleci.com).

I started with [ino](https://github.com/amperka/ino), but unfortunately the last commit was in 2014 and it no longer functions with the latest version of Arduino IDE.

Then I tried using the [arduino-cli](https://github.com/arduino/arduino-cli) to build a [Makefile (GitHub)](https://github.com/garyjohnson/build-button/blob/cf1c7cafbe215add65a0cb004e41e90da2040dd3/firmware/Makefile) but couldn't quite get it functioning with the Adafruit nrf52.

Might come back and revisit this later, but for now I'm going to have to let this go.

## Next Steps

It's time to design an enclosure!
