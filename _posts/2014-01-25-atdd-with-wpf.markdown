---
layout: post
title:  "ATDD with WPF"
date:   2014-01-25 22:05:20
category: articles
published: true
image:
  feature: background-02.png
---

## Introduction
As I was ramping up on a new WPF project, I knew I wanted to apply [ATDD concepts](http://powersoftwo.agileinstitute.com/2013/01/the-sportscar-metaphor-tdd-atdd-and-bdd.html) to validate our application against business and user requirements, and I wanted to validate these requirements against the *whole* application from the UI down. I've done this to great success in iOS and Android, but in WPF apps traditionally UI testing has been challenging. There are few options for testing WPF apps from the UI:

### Coded UI Tests
Coded UI Tests are a way to record and playback UI actions and validate UI output. Let me be clear. I hate, hate, hate Coded UI Tests and do not think you should use them under any circumstance. 
Why? 

- They are brittle and very dependent on the structure of your UI not changing.
- They produce a lot of auto-generated code that is not pleasant to look at or change.
- They require Visual Studio Premium or Ultimate, both expensive SKUs. No Express edition, no msbuild on your CI server. Sorry, but the testing framework you rely on should be available to everyone at least and open-source at best.

### Third-Party Tools
Telerik has their [Test Studio](http://www.telerik.com/teststudio/wpf-testing) product, but again I think something as critical as testing should rely on tools that are available to everyone, so I haven't given them much thought.

### SpecFlow
[SpecFlow](http://www.specflow.org/) is pretty rad. It's a Gherkin parser with Visual Studio integration and runs against C# unit tests. While cucumber is a great tool, it does require a little Ruby, and I like to keep the tests and the project in the same language when possible. This makes SpecFlow a nice option in many cases. The only problem is that it's not applicable here because it can't actually drive our UI.

### Not Testing UI At All
Okay, so there really aren't a lot of options for UI-level testing in WPF. What I usually fell back on was test-driving the ViewModel and everything behind it with unit tests, conveniently ignoring the View. No, you can't really test UI elements in unit tests, so that's out too. This is exactly the approach I wanted to avoid coming into a new project.

## Enter Mohawk
Once you start using cucumber to run your acceptance tests, everything starts to look like a nail (that's the expression, right?). I reached out to [Levi Wilson](https://twitter.com/leviw) to see what he was up to and it turns out he had authored a solution that was pretty much exactly what I was looking for: an adapter that allowed you to access UI Automation from cucumber, and do it with a minimal amount of code.     

![](/images/mohawk1.png)   
   
## Getting Started
I'm going to walkthrough validating a simple WPF application against acceptance tests using [cucumber](http://cukes.info/) and [mohawk](https://github.com/leviwilson/mohawk). 

### Requirements
[Ruby](http://rubyinstaller.org/downloads) (I used 1.9.3, but feel free to experiment with 2.0)   
[Ruby Dev Kit](http://rubyinstaller.org/downloads/) (Get the one that matches your version of Ruby)    

* Install Ruby
* Unpack Ruby Dev Kit into a directory (this is where it will live, so pick a good one)
* In cmd or powershell or whatever console you use, run (from the Dev Kit directory):   

{% highlight bat %}
ruby dk.rb init   
ruby dk.rb install
{% endhighlight %}

Okay, let's talk about your console for a minute. We're going to want that cucumber output looking pretty in Windows. Unfortunately cmd.exe does not do colors by default. You've got a couple options. If you're really in love with cmd.exe, you can install [ANSICON](http://adoxa.hostmyway.net/ansicon/) and get ANSI colors in cmd.exe. However, I recommend you check out [ConEmu](http://www.hanselman.com/blog/ConEmuTheWindowsTerminalConsolePromptWeveBeenWaitingFor.aspx). It's great, it handles ANSI colors, you can have it appear fullscreen with a single keystroke. What more could you want?

* Install [Bundler](http://bundler.io/). Bundler is sort of, kind of the Ruby analog of NuGet for you .NET folks. You can install it with the following command:

{% highlight bat %}
gem install bundler
{% endhighlight %}

### Writing our acceptance tests
Okay, we're ready, let's get into it. First off, all of this code is available at [github.com/GaryJohnson/wpf_atdd](https://github.com/GaryJohnson/wpf_atdd), so if you get lost or flub something up (or if I didn't write very good instructions), you can check your work against that repo.

Create your project directory, and in the root create a file named "Gemfile" (no extension) so Bundler will know what dependencies to install. Gemfile should contain the following text:

{% highlight bat%}
source "https://rubygems.org"

gem "cucumber"
gem "mohawk"
gem "ffi", "1.3.0"
gem "rspec-expectations"
{% endhighlight %}

**ffi** is a mohawk dependency that we're locking into a specific version due to an issue at the time of writing. Feel free to try removing it. [**rspec-expectations**](https://github.com/rspec/rspec-expectations) is not necessary, but we're including so we can have nice *should* style assertions.    
    
Next, install your dependencies that you just added to the Gemfile:    

{% highlight bat %}
bundle install
{% endhighlight %}

Okay, now time to write some Gherkin for our acceptance tests. In the project root, create a **features** directory, and a **step_definitions** directory inside that. In the features directory, create a text file called **login.feature**. Let's add our acceptance test:

{% highlight gherkin %}
Feature: Logging In
  In order to test ATDD in WPF
  As a developer
  I want to make sure this actually works

  Scenario: Test
    When I log in as 'Tom'
    Then I should see 'Hello Tom'
{% endhighlight %}
