---
title: "The Most Important Thing"
authors: ["gio"]
date: 2017-01-14T20:26:00+07:00
tags:
- software engineering
- testing
---

> If you were to be stranded on a deserted island, and to escape from the said island you have to build a kick-ass web or mobile app using any language and environment that you prefer, what would you prioritize?

You also got a team to help you. They're supposed to be badass, but you don't know them before and never work with them.

![The Team](/images/the_team.jpg)
<small>*This is your team, they are badass but you're not familiar with their skillz*</small>

Okay, that was a really cheesy way to make an entrance. Let me try again.

The point that I'm trying to make was, for me, the most important thing that absolutely cannot be left out in software development is **testing**.  

This is another strong statement from me, but I believe testing is what separates *professional* software development and *just* software development (whatever "professional" means, but it sounds, well, professional). It is also what makes software engineer a *proper engineer* (lol). 

However testing is a time-consuming, (in some cases) tedious and so initially unrewarding that non-tech manager in a tight deadline situation would most probably say: *All hands abandon test! one person one feature! every man for himself!*

## Techniques

The most basic way to test your application is to ask someone to access it and ask them to try whatever they want with it until it breaks (or not). But it's not very scientific so I'm not gonna pursue this topic further.

> It happens that I still see instance where boss tell their staff to test an application by using said method. Usually they will input some improbable values to be inputted in real-world situation or doing some improbable actions. They also doing so without consulting a test document. I dislike this approach as it will increase burden for the developer to fix improbable-to-happen bugs, especially without test document to back-up.

![Riot](/images/riot.jpg)
<small>*Ask someone to break your application*</small>

The next thing that is more scientific is to prepare a **test document** to be used as guidance in testing. This is perhaps a bit outdated but it's still quite effective in some cases. For example, I haven't been able to get a good coverage in automated front-end integration testing, therefore I'll write down all critical scenarios that is affected by changes to be tested by QA's before deployment. 

> One thing though, front-end integration testing is an interesting topic that I want to discuss further in my future writings (or maybe if anyone reading this has ideas, please chime in. I do encounter difficulties in this area).

Lastly, there is also automated testing using **test framework**. This is the best thing that you can (must) utilize when you want to test your application. Test framework has actually been around for quite some time (see [this](https://shebanator.com/2007/08/21/a-brief-history-of-test-frameworks/) post for history of JUnit, which is one of Java test framework).

If I have to put it simply, utilizing test framework to do automated testing is like writing test document and ask QA's to test it. However, you must change "test document" into "structured document" (so that the test framework understand it) and you must also change "QA" with "your computer". 

The thing is, this "structured document", which previously can be written by system analyst, business analyst, QA or programmer itself can now only be written by programmer. This is because the structured document is practically **codes**. There's been effort to "humanize" it by project, such as [Cucumber](https://cucumber.io), but I feel that there are still some gaps to be addressed to make non-programmer write test themselves.

I now have touch the general techniques for testing, before continuing I want to make a small detour first to discuss about the *Invisible Reward*.

## Invisible Reward 

Here is the conundrum with software testing, if you want to get a reward with testing you have to do it **consistently** and **thoroughly**. To do it thoroughly, you will need the commitment from everyone: your team, your boss and/or your customer and everyone should prepare to give substantial amount of time, usually spent for feature development, to testing. 

Remember that, if you do it half-heartedly, you won't get any benefit from testing. And if you are not doing it consistently your test artifacts and your codes won't be compatible thus making it useless (also waste time).

I did said that half-hearted testing won't give you reward, but the funny thing is even if you do it consistently and thoroughly, not everyone will get the reward (at least directly).

Okay, I fess, the only one who will get the reward directly is programmer. But the benefits are substantial and it will eventually trickle down to everyone and to the project as a whole.

These are the benefits of doing proper testing on your software development project:

1. Deploy without fear
2. Refactor without fear
3. Adding features without fear
4. Give feedbacks to programmers
5. Easier on-boarding of new programmers
6. Reduce the chance of someone to break the application
7. Self-documented codes
8. Can setup monitoring system or continuous integration system (when doing automated testing)
9. Feeling of security for everyone involved in the project (especially when you show them the cool CI dashboard, business people dig it)

I wrote the above list without any references. It was purely what I had felt and experienced about testing when I wrote this.

## Recipe for Effective Testing

If I want to talk about effective software testing, then I have to first introduce metrics that we can use to measure whether our tests are effective or not. I use two metrics myself, the first one is **test coverage** and the other one is **feedback loop time**. I'm sure there are many more, so maybe you can share it with me on how you do it on your place.

First, I will talk about test coverage. Test coverage means, the percentage of your codes that are tested (either automated or manual by QA). Higher test coverage means more part of your software are tested, which is better. 

So that probably means we should get a 100% test coverage right? No, in my opinion **100% test coverage is overkill**. Because higher test coverage also means more test codes and documents to be written, which takes time. Instead of 100%, it is more effective to set the bar at around 80% test coverage in which critical features must be included in the 80%.

> To calculate the percentage you may use the number from test framework, they usually display it after you run your test suite. You can also list all stories and determine which stories have been tested and which stories haven't

The next metric is feedback loop, it is the term that we can use to describe the relationship between writing codes, testing and evaluating the test result.

![Feedback Loop](/images/feedback_loop.jpg)
<small>*Feedback Loop*</small>

Accordingly, feedback loop time is how long does it takes from writing codes until the test result gets back to the programmer again. It will be great if we can **keep the feedback loop time short**, so that programmer can make repair as soon as possible. Long feedback loop time is harmful. The reason is because your programmers can already be working on other things when they got the feedback. They will also forget about what they had done that warrant said feedback as time goes by.

One additional thing to consider regarding feedback loop: if we decided to do automated testing, the feedback loop will be very short (which is great!). We can also utilize a continuous integration and/or a monitoring system to further enhance the speed and information distribution so that all appropriate parties can get information faster and properly if something make or break in our software. 

## The Windup

That is all for now, I now conclude that by properly adjust and monitor test coverage percentage and feedback loop time we can control and measure the effectiveness of our testing activity. 

I feel that some topics in this writing can be explored further, but I will save it for later. 

Ciao.
