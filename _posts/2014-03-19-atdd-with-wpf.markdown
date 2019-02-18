---
layout: post
title:  "ATDD with WPF"
date:   2014-03-19 22:05:20
category: articles
published: true
image:
  feature: background-02.png
---

## Introduction
As I was ramping up on a new WPF project, I knew I wanted to apply [ATDD concepts](http://powersoftwo.agileinstitute.com/2013/01/the-sportscar-metaphor-tdd-atdd-and-bdd.html) to validate our application against business and user requirements, and I wanted to validate these requirements against the *whole* application from the UI down. I've done this to great joy in iOS and Android, but in WPF apps traditionally UI testing has been challenging. There are few options for testing WPF apps from the UI:

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
Once you start using cucumber to run your acceptance tests, everything starts to look like a nail (that's the expression, right?). I reached out to [Levi Wilson](https://twitter.com/leviw) to see what he was up to and it turns out he had authored a solution called [mohawk](https://github.com/leviwilson/mohawk) that was pretty much exactly what I was looking for: an adapter that allowed you to access UI Automation from cucumber, and do it with a minimal amount of code.     
![](/images/mohawk1.png)   
   
## Getting Started
I'm going to walk through validating a simple WPF application against acceptance tests using [cucumber](http://cukes.info/) and [mohawk](https://github.com/leviwilson/mohawk). 

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

## Writing Acceptance Tests
Okay, we're ready, let's get into it. First off, all of this code is available at [github.com/GaryJohnson/wpf_atdd](https://github.com/GaryJohnson/wpf_atdd), so if you get lost or flub something up (or if I didn't write very good instructions), you can check your work against that repo.
<br/><br/>
Create your project directory, and in the root create a file named "Gemfile" (no extension) so Bundler will know what dependencies to install. Gemfile should contain the following text:

{% highlight bat%}
source "https://rubygems.org"

gem "cucumber"
gem "mohawk"
gem "ffi", "1.3.0"
gem "rspec-expectations"
{% endhighlight %}

**ffi** is a mohawk dependency that we're locking into a specific version due to an issue at the time of writing. Feel free to try removing it. [**rspec-expectations**](https://github.com/rspec/rspec-expectations) is not necessary, but we're including so we can have nice *should* style assertions.    
<br/><br/>
Next, install your dependencies that you just added to the Gemfile:    

{% highlight bat %}
bundle install
{% endhighlight %}

Okay, now time to write some Gherkin for our acceptance tests. In the project root, create a **features** directory, and inside features create **step_definitions**, **support**, and **page_objects** directories. In the features directory, create a text file called **main_page.feature**. Let's add our acceptance test to this file: 

{% highlight gherkin %}
Feature: Logging In
  In order to test ATDD in WPF
  As a developer
  I want to make sure this actually works

  Scenario: Test
    When I log in as 'Tom'
    Then I should see 'Hello Tom'
{% endhighlight %}

Of course, this doesn't actually do anything yet without code. Let's run cucumber from the project root:

{% highlight bat %}
cucumber
{% endhighlight %}

You should see output similar to this:

<br/>![](/images/mohawk2.png)<br/>

## Writing Step Definitions

Armed with some examples of empty step defintions, let's copy those into our steps file. In **features\step_definitions**, create a text file named **main_page_steps.rb** and paste in the examples from the output we just got:  

{% highlight ruby %}
When(/^I login as 'Tom'$/) do
  pending # express the regexp above with the code you wish you had
end

Then(/^I should see 'Hello Tom'$/) do
  pending # express the regexp above with the code you wish you had
end
{% endhighlight %}

In each of those steps we now need a way to actually talk to the UI (the UI we haven't created yet). This is where mohawk comes in. Mohawk works using the concept of [page objects](http://www.cheezyworld.com/2010/11/19/ui-tests-introducing-a-simple-dsl/), which, to over-simplify, is a pattern where we create an abstract for the code that identifies and communicates with the UI.
<br/><br/>
Let's create a page object for our only page. In **features\page_objects** create a text file named **main_page.rb**. First we need to reference mohawk, so start with

{% highlight ruby %}
require 'mohawk'
{% endhighlight %}

Now our class, and include members from the **Mohawk** module:
{% highlight ruby %}
require 'mohawk'

class MainPage
  include Mohawk

end
{% endhighlight %}

Now we need a way to identify the main window so that mohawk can find it. In our case, the window title is going to be "MainWindow", so let's use that:
{% highlight ruby %}
require 'mohawk'

class MainPage
  include Mohawk
  window(:title => 'MainWindow')

end
{% endhighlight %}

We need to access three things to make this work -- we need to input a user name into a textbox, press a log in button, and then inspect a label to make sure it changes to what we want. 

{% highlight ruby %}
require 'mohawk'

class MainPage
  include Mohawk
  window(:title => 'MainWindow')

  text(:username, :id=> "UserName")
  button(:login, :value => "Log In")
  label(:message, :id => "Message")
end
{% endhighlight %}

Okay! So we have an abstraction for our UI. Here's what that UI looks like, by the way:

<br/>![](/images/mohawk3.png)<br/>
<br/>
Let's go back to our step definitions and fill them out a little. Let's start with our When statement:

{% highlight ruby %}
World(Mohawk::Navigation)

When(/^I login as '(.*)'$/) do |name|
  on(MainPage) do |page|
    page.username = name
    page.login()
  end
end
{% endhighlight %}

Okay, so there's some metaprogramming magic going on here. Remember above in our page object when we declared:

{% highlight ruby %}
  text(:username, :id=> "UserName") #created page.username
  button(:login, :value => "Log In") #created page.login()
{% endhighlight %}

text(:username) created **page.username**, and getting page.username will return the string in the textbox with id="UserName", while setting it will write text to the textbox.
<br/><br/>
button(:login) acts a little differently. It created **page.login()**, which is a method that will press a button on the screen containing a value of "Log In".
<br/><br/>
button() and text() are called accessors. There are accessors for each different type of control such as label, link, button, table, and they all act a little differently depending on how you might interact with them. Check out [accessors.rb](https://github.com/leviwilson/mohawk/blob/master/lib/mohawk/accessors.rb) to see all the different types and what members they create on the page object.
<br/><br/>
Now time for our Then statement:

{% highlight ruby %}
Then(/^I should see '(.*)'$/) do |message|
  on MainPage do |page|
    page.message.should eq message
  end
end
{% endhighlight %}

Okay, so in our page object, label(:message) created **page.message**, which we can read for a string. The **.should eq** comes from [rspec-expectations](https://github.com/rspec/rspec-expectations) and will fail the test if our UI doesn't match the message passed into the step.
<br/><br/>
Finally, you need to set up your env.rb. Create or open the file **/features/support/env.rb**. It needs to look like this:

{% highlight ruby %}
require 'mohawk'
require 'rspec/expectations'

Mohawk.app_path='TestApp\bin\Debug\TestApp.exe'

Before do
  Mohawk.start
end

After do
  Mohawk.stop
end
{% endhighlight %}

This will make sure Mohawk launches your app before each test, and quits after each test.
<br/><br/>
We're there! Go ahead and run cucumber from the project root:
{% highlight bat %}
cucumber
{% endhighlight %}

You should see the tests pass:

<br/>![](/images/mohawk4.png)<br/>
<br/>
Of course, this is backwards. We should be starting with failing tests. Let's make it fail! Open up **/TestApp/MainWindow.xaml.cs** and change the line that updates the message:

{% highlight c# %}
Message.Text = string.Format("Goodbye {0}", UserName.Text);
{% endhighlight %}

Save and build(!) the app in Visual Studio, then run cucumber again:

<br/>![](/images/mohawk5.png)<br/>
<br/>
Hopefully that got you started with mohawk in WPF. From here, I would set up env.rb to execute msbuild so we don't have to manually build our project with each run, but that's an exercise I leave up to you.
<br/><br/>
Happy testing!
