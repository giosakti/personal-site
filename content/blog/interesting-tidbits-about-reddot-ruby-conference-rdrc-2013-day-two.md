---
title: "Interesting Tidbits about Reddot Ruby Conference (RDRC) 2013 - Day Two"
authors: ["gio"]
date: 2013-07-24T12:42:00+07:00
tags:
- conference
- ruby
---

This is a continuation from my previous post regarding Reddot Ruby Conference 2013 A.K.A RDRC last June in Singapore. After discussion of Aaron ([@tenderlove](https://twitter.com/tenderlove‎)), Luismi ([@cavalle](https://twitter.com/cavalle‎)) and Jim ([@jimweirich](https://twitter.com/jimweirich)) talks in [day one](/blog/interesting-tidbits-about-reddot-ruby-conference-rdrc-2013-day-one/), now we move onto day two.

*You can also check RDRC official website [here](https://www.reddotrubyconf.com/)*

## Day Two

#### Jose Valim - Concurrency in Ruby

Our morning was greeted by another high-profile speaker, Jose Valim. Most of you may recognized him from gems that he authored: [devise](https://github.com/plataformatec/devise) and [simple_form](https://github.com/plataformatec/simple_form). Other than those gems, Jose also works as Rails Core Team member. He is number #4 in Rails contributors of all time [standing](https://contributors.rubyonrails.org/contributors) (Aaron from day one is number #3).

Jose topic was about **concurrency in Ruby** and **tools of the trade** that we can utilize to implement it. During the beginning of his presentation, Jose told us that he just gives a talk with the same topic but with slightly different content during RubyKaigi event a week before in Japan. He suggested that we should also check his [slides](https://speakerdeck.com/plataformatec/concurrency-in-ruby-in-search-of-inspiration) from that event. Unfortunately, I haven't got the time to compare his slides from RubyKaigi with his slides from RDRC.

Concurrency, is a very interesting topic not just for Ruby, but for programming in general. One of the primary reason is because the current trend in hardware technology advancement is not the same as it was in the past. During late 90s to beginning of 00s, it was all about clockspeed, mega/gigahertz war and so on. Now it was about multiple threads and multiple cores. Therefore we must change the way we program to utilize all the threads and all the cores that modern hardware can provide.

**What is Concurrency?**

Jose illustrate concurrency by showing some codes and walk us through it. The codes can be seen below.

```ruby

# Example of concurrency

class User
  def self.current_user=(user)
    @@current_user = user
  end

  def self.current_user
    @@current_user
  end
end

post "/do_something" do
  User.current_user = 
    user_from_request

  # some work

  Mailer.deliver_to(
    User.current_user)
end
```

Basically, the code was about sending e-mail everytime a request was made to `/do_something`. The E-mail is sent to whomever user currently registered on `User.current_user` variable. It's a class variable.

The inherent concurrency problem may not be visible from the code itself, however Jose gives us a nice accompanying illustration to demonstrate where the complication arise.

<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/concurrency-problem-illustration.jpg" alt=""> -->

The problem exist because `User.current_user` is a shared mutable state (A.K.A global variable) and there were multiple process trying to read/write `User.current_user` in a way that disturb each other process. On this case, thread 2 is making a request in almost the same time as thread 1. Therefore both are writing `User.current_user` before the other process finished sending the e-mail, making the process send the e-mail to the wrong user.

This is one of the reason on why you often see advice talking about global variable is a bad thing. However, you will encounter a situation where you have no option other than to utilize global variable. In fact, it was utilized even in Rails or other popular gems that we use. Examples:

1. `ActionMailer::Base.from`
2. `ActionController::Base.logger`
3. `Devise.password_length`
4. `SimpleForm.form_options`

**Current State for Concurency in Ruby**

At present time, Ruby support for concurrency was rather "tricky". This is because different ruby implementation yield different behavior. For example MRI 1.8 has green threads for handling concurrency (threads that were created/scheduled by vm) therefore it wasn't really concurrent. MRI 1.9 tried to address it by having real threads (threads that were created/scheduled by os). However due to the implementation of GIL (global interpreter lock or global vm lock - GVL as ruby called it), it still only allows one thread to execute at a time. JRuby, on the other hand, has real threads and can fully support concurrency.

The situation has impacted many things that you have to address, especially if you utilize multiple ruby implementations at a time. Let's take for example Rails, in Rails we have `config.threadsafe!` option to enable thread-safety, which in turn allow Rails to utilize multithreaded environment. 

However, due to the difference in each ruby implementation, **thread-safety means different things for different implementations**. Because MRI has GIL/GVL, it won't allow those threads to execute in parallel thus avoiding concurrency problem that we have discussed in the previous section. However JRuby doesn't have GIL/GVL and those problems may still occur if it's not handled properly.

**Introduction to the tools of the trade**

To avoid the problem, we have to modify our application codes so that it can avoid concurrency problem regardless of ruby implementation that we use. These are some of the options that Jose introduced to help in improving our codes.

**Thread locals**

Thread locals is about storing the variables in the current thread with the help of `Thread.current` method.

```ruby

# Not using thread locals

class User
  def self.current_user=(user)
    @@current_user = user
  end

  def self.current_user
    @@current_user
  end
end

# Using thread locals

class User
  def self.current_user=(user)
  Thread.current[:cu] = user
  end

  def self.current_user
  Thread.current[:cu]
  end
end
```

As we can see above, rather than storing our variable in a global (class) container, we can store it in a local thread hash container.

**Eager loading**

The problem with ruby autoload function is that they are not thread-safe. This is a technique that could be used to remedy that situation by ensuring the required libraries are loaded up-front during booting.

```ruby
module MyApp
  extend ActiveSupport::Autoload

  eager_autoload do
    autoload :Broker
  end
end

MyApp.eager_load!
end
```

**Mutex**

Mutex is one of the "classic" solution for handling concurrency. Mutex or mutual exclusion is a "semaphore" that has the ability to coordinate access to shared data across concurrent threads. Ruby has [mutex](https://www.ruby-doc.org/core-2.0/Mutex.html) library that we can utilize.

```ruby

# Not using mutex

hash = {}

10.times do |i|
  Thread.new {
    hash[i] = "Hello #{rand}"
  }
end

# Using mutex

hash = {}
mutex = Mutex.new

10.times do |i|
  Thread.new {
    mutex.synchronize {
      hash[i] = "Hello #{rand}"
    }
  }
end
```

**Thread_safe gem**

[headius](https://github.com/headius) has an interesting gem called [thread_safe](https://github.com/headius/thread_safe), which contains "collection of utilities for thread-safe programming in Ruby".

Some of the examples are thread-safe `Hash` and thread-safe `Array`.

**Better tools of the trade**

Other than tools of the trade that were described above, Jose also points out that **Queue** maybe the better solution that can address concurrency problem. Although I think more effort are required to implement queueing system in our codes.

He also mentions [Celluloid](https://github.com/celluloid/celluloid). I haven't checked it, but based on the description that was provided by the author (and the number of stars it received :D), I think it's very promising.

*You can check Jose's slide [here](https://speakerdeck.com/plataformatec/concurrency-in-ruby-tools-of-the-trade)*

#### Akira Matsuda - Ruby 2.0 on Rails in Production

Akira Matsuda was also a keynote speaker in the 2nd day. As one of Ruby core developer, he promoted ruby 2.0 as ruby that is "already ready for production" (unlike ruby 1.9.0 before).

There are four interesting topics that he gives to influence us to migrate to ruby 2.0: **performance**, **stability**, **compability** and **new features**.

**Performance**

Ever heard of Rails application that has **1,003 models**, **236 controllers** and **2871 view templates**? meet [Cookpad](https://cookpad.com). A popular cooking application in Japan that turns out to be a very (if not the most) complex Rails application. Akira told us that he has the opportunity to work on cookpad.com for a while, and someone tried to compare its performance when running on different ruby version.

The result can be seen below :)

<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/ruby-2.0.0-vs-ruby-1.9.3.jpg" alt=""> -->

I have to agree with Akira that this application is an ideal example to test for performance.


**Stability**

Straight from the start, ruby 2.0.0 already has patchlevel-195 and touted as a "stable" release from the beginning by the ruby team. It is very encouraged to move to this version as soon as possible, because support for older rubies will be cut-off eventually.

**Compability**

Ruby 2.0 is virtually "100% compatible" with previous ruby release, other than some exceptions. By default, Ruby 2.0 now use utf-8 as source file encoding, so you may freely use utf-8 character within your source code.

<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/ruby-2.0.0-gc-bug.jpg" alt=""> -->

**New Features**

Ruby 2.0 introduced dozens of refinements and bugfixes that will enhance our experience in working with ruby. `Module#prepend` is also a good addition, we can use this method to replace `alias_method_chain` (a rails library). Although, truthfully, I haven't got the opportunity to use this method in any of my application.

The new **keyword arguments** feature is an excellent addition. While the usual method parameters were behaving like an array, thus making the order important, the new method parameters are more like a hash, which means that order isn't important anymore and you can assign each parameter by using a label.

```ruby
# traditional method
def m(a, b = 'b', c = 'c') end

m('1', '2', '3')

# keyword arguments A.K.A kwargs
def m(a, b: 'b', c: 'c') end

m('1', b: '2', c: '3')
```

I already see the potentials in improving the readability of my codes using kwargs :) However, important notice from Akira: **do not use 'if', 'unless' and 'end' as an argument name..**

Other important addition that worth mentioning is `Enumerable#lazy`. It enables us too iterate "over an infinite series of values and take just the values you want".

```ruby

# This will perform flawlessly

range = 1..Float::INFINITY
p range.lazy.collect { |x| x*x }.first(10)
```

*You can check Akira's slide [here](https://speakerdeck.com/a_matsuda/ruby-2-dot-0-on-rails-in-production)*

#### Simon Robson - Optimising Self


> The programmer, like the poet, works only slightly removed from pure thought-stuff. He builds his castles in the air, from air, creating by exertion of the imagination.  
> Fred Brooks - The Mythical Man-Month

Keeping inline with RDRC traditions, this year conference also introduce non-technical (and often, philosophical) topic that helps us improve as a programmer. I really liked Simon Robson talks this year regarding "Optimising Self". Simon is a programmer whom have his own startup company in Chiang Mai, Thailand.

Well, obviously self in this topic means us as a programmer. Simon begins by questioning us on **why?** we as programmers must improve.

> My ongoing and future personal comfort depends on my ability to design and realize creative works which meet certain externally imposed requirements.

A very good answer and very concise of my current situation :) 

Now, To improve ourselves, Simon identify three important goals: **Promote creativity - prevent burnout**, **promote technical skill - prevent stagnation** and **promote longevity - prevent death**. Each goal was then discussed in details: what are the things that may prevent us from reaching the goal and what should we do to take us closer to the goal.

**Goals**

**Promote creativity - Prevent burnout**  
- work too hard / too long on the same thing  
- too much context switching  
- working on distasteful things (in ethical / engineering terms)  
+ take a break / go to bed  
+ one day, one or two things - plan & schedule  
+ say no  
+ fix the (engineering) pain  

**Promote technical skill - prevent stagnation**  
- doing the same thing time and again  
- multiple projects, same requirements  
- lack of challenges  
- lack of new things  
+ solve repetition with engineering  
+ make space (i.e. time) for learning  
+ start a project. or join an existing project  

**Promote longevity - prevent death**  
- disease  
- stress  
- accidents  
- old age  
+ get exercise. Make it routine  
+ recognize stress. And look it in the eye  
+ Don't text and drive / walk and read / etc  
+ age like wine  

<!-- <img src="http://mightygio.com/wp-content/uploads/2013/07/optimising-self.jpg" alt=""> -->

**Overcoming the lizard brain**

> The resistance grows in strength as we get closer to shipping, as we get closer to an insight, as we get closer to the truth of what we really want. That's because the lizard hates change and achievement and risk.  

> [Quieting The Lizard Brain](https://sethgodin.typepad.com/seths_blog/2010/01/quieting-the-lizard-brain.html)

Simon also touch a topic regarding the "lizard brain". Lizard brain is a terminology that was mentioned by [Seth Godin](https://www.sethgodin.com/), described as an **irrational human behavior that occurs when we say something and do the opposite instead**. For example (taken from Seth's essay): We say we want one thing, then we do another. We say we want to be successful but we sabotage the job interview. We say we want a product to come to market, but we sandbag the shipping schedule. We say we want to be thin but we eat too much. We say we want to be smart but we skip class or don't read that book the boss lent us. 

To address this behaviour of ours, Simon shared some important tips for us:

**1. Tackle the difficult thing right now**  
  Upfront analysis easily turns to procrastination
  Dive in, get hands dirty then do some analysis

**2. Plunge headfirst into the new**  
  Conquer fear of messing up / looking dumb
  You will anyway. Get over the hump quicker

**3. Disable autopilot**  
  Typing isn't the core of your job
  ALmost certainly a learning experience to be had

**4. Share early**  
  Failure looks worse the longer you hide it
  Involving others reduces chance of failure

*You can check Simon's slide [here](https://speakerdeck.com/shr/optimising-self-at-reddotrubyconf-2013)*

## Other Interesting Talks

Rdrc 2013 has dozens of interesting talks, unfortunately I can't cover them all in my writings :) These are some of the talks that I also find interesting.

**Ola Bini - JRuby for the Win (Day One)**

Ola Bini is one of [JRuby](http://jruby.org/) core developers. He took the opportunity on stage to explain all about JRuby: what is it? how it compares with other ruby implementations? and all the advantages that it had against other implementations. I tried JRuby on some of my projects and I found the ability to call Java library from within Ruby is very amusing. 

You can check Ola's slide [here](http://winstonyw.com/assets/downloads/JRubyForTheWin.pdf)

**Alexey Gaziev - Rails and Javascript: Brothers in Arms Using Gon (Day One)**

Alex is programmer from Russia who works remotely in Bali. I've happened to came across him during the conference and talks a bit with him. [Gon](https://github.com/gazay/gon) is one of his creation that I've used previously in some of my projects. Gon helps in seamless serialization between javascript and Rails, while also introduces "DRY"er code and live reloading.

You can check Alexey's slide [here](https://www.slideshare.net/gazay/gon-RDRC)

**Steve Klabnik - Functional Reactive Programming in Ruby (Day Two)**

Steve was another household name that came and give talks during RDRC 2013. He talks about program writing styles in Ruby that is interesting, but haven't been used in Ruby at all. This style of writing program is called 'Functional Reactive Programming' a style that is often used in functional language, such as Haskell.

## Conclusion

Okay then, that was it for RDRC this year. It was truly an interesting and inspiring event. Thanks to [@winstonyw](https://twitter.com/winstonyw), [@mohangk](https://twitter.com/mohangk), [@abhayashenoy](https://twitter.com/abhayashenoy), [@luweidewei](https://twitter.com/luweidewei) for organizing it and [@andycroll](https://twitter.com/andycroll) as the initiator. 

For next year, I'm also planning to go to [RubyKaigi](https://rubykaigi.org/) with my friends from [id-ruby](http://id-ruby.org/) (Indonesia Ruby Community) if our schedules match with the event. 

Hope you all enjoy my post :)
