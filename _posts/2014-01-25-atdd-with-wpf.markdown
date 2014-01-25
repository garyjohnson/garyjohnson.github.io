---
layout: post
title:  "ATDD with WPF, Part 1"
date:   2014-01-25 22:05:20
categories: wpf atdd mohawk
---

## Introduction
As I was ramping up on a new WPF project, I knew I wanted to apply ATDD concepts to validate our application against business and user requirements. I've done this to great success in iOS and Android, but in WPF apps traditionally UI-level testing has been challenging. 

## Existing Approaches

### Coded UI Tests
Coded UI Tests are a way to record and playback UI actions and validate UI output. Let me be clear. I hate, hate, hate Coded UI Tests and do not think you should use them under any circumstance. Why? 
- They are brittle and very dependent on the structure of your UI not changing.
- They produce a lot of auto-generated code that is not pleasant to look at or change.
- They require Visual Studio Premium or Ultimate, both expensive SKUs. No Express edition, no msbuild on your CI server. Sorry, but the testing framework you rely on should be available to everyone at least and open-source at best.

### Not Testing UI
Okay, so there really aren't a lot of options for UI-level testing in WPF. What I usually fell back on was using the MVVM pattern and test-driving the ViewModel and everything behind it with unit tests, conveniently ignoring the View. No, you can't really test UI elements in a unit test context. 

## Mohawk
Once you start using cucumber to run your acceptance tests, everything starts to look like a nail (that's the expression, right?). I reached out to [Levi Wilson](https://twitter.com/leviw) to see what he was up to and it turns out he had authored a solution that was pretty much exactly what I was looking for: an adapter that allowed you to access UI Automation from cucumber.

