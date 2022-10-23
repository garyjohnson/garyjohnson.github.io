---
layout: single
title:  "Bluetooth Build Button, Part 7: crashes and animations"
date:   2019-05-11 10:29:00
author: garyjohnson
category: articles
published: true
header:
  image: /images/button07/cover.jpg
---

Previously, we [added LEDs for feedback (Part 6: lights and a power switch)]({% post_url 2019-03-19-lights-and-a-power-switch %}), and ended up with a number of problems to fix. The most blocking was an issue where making changes to the firmware started causing inexplicable crashes.

## Crashes

While working on animation code, I ran into an issue with the firmware where adding doing seemingly trivial things in the code (declaring new unused variables or adding empty methods to a class for example) would compile but result in completely non-functional firmware. After flashing the firmware, the board LED would blink red a couple times and then become unresponsive.

I took a couple troubleshooting steps: could I be out of memory? Checked the global memory usage output by the compiler and also at runtime (with a functioning version of the firmware) and there appeared to be pleny of free memory.

A friend suggested the possibility of defective hardware (a bad block in memory where the firmware is stored, maybe), but I had some extra, identical boards to test on and they had the same issue.

Another possibility could be something in the toolchain, but debugging that felt like a total rabbit hole.

My next step was going to be ordering a [JTAG debugger](https://www.adafruit.com/product/3571) and seeing if I could at least sniff out some more detailed error info and enable better debugging. However, at some point in this process Adafruit released a new version (0.10.1) of [Adafruit_nRF52_Arduino](https://github.com/adafruit/Adafruit_nRF52_Arduino/releases), which (I think) is the toolchain, bootloader, and libraries that enable programming the Feather from the Arduino IDE. After upgrading, all my issues dissapeared. I'm not sure what exactly was broken, but I'll take it as a lucky win. 

Onward, to animations.

## Animations

One problem encountered during use is feedback -- once I release the button, the LEDs turn off immediately. It can be a little confusing to know what stage I just triggered (run test, run file, run all tests), but mostly it's just unsatisfying.
<br>
<video controls="controls" name="no-feedback" src="/images/button07/no-feedback.mp4" poster="/images/button07/no-feedback-thumbnail.jpg" preload="auto"></video>
<br>

### An End to Delays

There's a couple ways to approach building a feedback animation. It's pretty common when starting with programming micros like an Arduino to encounter code like this:


{% highlight cpp %}{% raw %}
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
{% endraw %}{% endhighlight %}

...in fact, this is from the Arduino built-in examples. And it's a fine approach if all you're doing is blinking an LED. What happens if you want to check for a button press?

{% highlight cpp %}{% raw %}
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
  if(digitalRead(BUTTON) == HIGH) {
    // do stuff
  }
}
{% endraw %}{% endhighlight %}

If you briefly press the button here, there's a high chance it won't work -- to be 100% sure you'll trigger your button handler, you'd have to hold the button down for > 2 seconds.

Rather than programatically defining our animation using `delay` to guarantee timing, let's _calculate_ our animation state at any given time:

{% highlight cpp %}{% raw %}
const unsigned int blinkDuration = 1000;

unsigned long previousMillis = 0;
unsigned long currentMillis = 0;

void loop() {
  currentMillis = millis();

  // current time divided by the duration of a blink (on or off) will
  // give us a "count" of how many animation phases have happened
  // ex: 3400 millis / 1000 blink phase = 3 phases
  unsigned long blinkPhase = currentMillis / blinkDuration;

  if(blinkPhase % 2 == 0) {
    // odd phase, light off
    digitalWrite(LED_BUILTIN, LOW);
  } else {
    // even phase, light on
    digitalWrite(LED_BUILTIN, HIGH);
  }

  if(digitalRead(BUTTON) == HIGH) {
    // do stuff
  }

  previousMillis = currentMillis;
}
{% endraw %}{% endhighlight %}
 
Look! No delays. A couple notes on how that example isn't perfect:

1. because we don't use the remainder of `currentMillis / blinkDuration`, there's a very slight delay to the animation on start.
2. At some point `millis()` will reach the max value that can fit in an `unsigned long` and rollover to zero -- we may see a hitch in the animation during that time (two seconds of being lit up, or two seconds of being off).
3. We rely on the simplicity of the animation being a two-state -- what if we want to fade the LEDs in and out or have multiple states?
4. The blinking always happens -- what if we want to trigger the blinking on button press, for a certain amount of time? What if we want to be able to interrupt it?

For a better and more detailed tutorial along these same lines, check out [Adafruit's 'multi-tasking the arduino'](https://learn.adafruit.com/multi-tasking-the-arduino-part-1/overview)

## Feedback Animation

It's with that approach in mind that I tackled feedback animations: [LedApp.cpp](https://github.com/garyjohnson/build-button/blob/f2ebf5d35bc040f05dc32b6827a9f14501823eb3/firmware/build-button/LedApp.cpp#L41-L59).

{% highlight cpp %}{% raw %}
pixels.setBrightness((sin((float)animationDuration/divisor) * 125.0f) + 130.0f);
{% endraw %}{% endhighlight %}

The real key in there is `sin()` -- we're using a sine function to ramp the brightness of the LEDs up and down in a smooth "blinking" fashion. The rest of the math is there to a) ensure the blinking is the right speed and b) ensure that the animation starts on the peak of the sine wave (we want to start the animation at full brightness, so that it appears to transition smoothly from the pushed state).

<br>
<video controls="controls" name="animation" src="/images/button07/animation.mp4" poster="/images/button07/animation-thumbnail.jpg" preload="auto"></video>
<br>

## What's Next

Every day I need to charge the button even when powered off -- it's time to rework the power switch. Also, the lid not "locking" into place is troublesome -- it pops off frequently.
