---
layout: post
title:  "ATDD with WPF, Part 1"
date:   2014-01-25 22:05:20
categories: wpf atdd mohawk
---

## Introduction
As I was ramping up on a new WPF project, I knew I wanted to apply ATDD concepts to validate our application against business and user requirements. I've done this to great success in iOS and Android, but in WPF apps traditionally UI testing has been challenging. 

## Existing Approaches

### Coded UI Tests
Coded UI Tests are a way to record and playback UI actions and validate UI output.

Let me be clear. I hate, hate, hate Coded UI Tests and do not think you should use them under any circumstance. Why? 
- They are brittle and very dependent on the structure of your UI not changing.
- They produce a lot of auto-generated code that is not pleasant to look at or change.
- They require Visual Studio Premium or Ultimate. No Express edition, no msbuild on your CI server.

