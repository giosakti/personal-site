---
title: "Rethinking Software Engineering"
authors: ["gio"]
date: 2017-01-07T16:48:00+07:00
tags:
- software engineering
---

Several weeks ago, I got to met my fellow ruby programmer friends in Jakarta. We're talking about various things, including discussion about the distinction between informatics engineering terminologies, such as: computer science, programming, software engineering, software developer and informatics engineering itself.

I vaguely know for a long time that each and every terminology is different, however slightly it was. Still, people are using some of it interchangeably, disregarding subtle difference between them (including myself, I knew they were different but I still did it anyway)

Considering that, I then peruse the web and found out in quora that many people ask the same question about the difference between them. Some of it are [here](https://www.quora.com/Whats-the-difference-between-a-software-engineer-and-a-programmer), [here](https://www.quora.com/What-is-the-difference-between-Programmer-coder-Software-Engineer-Manager), [here](https://www.quora.com/Coding-Staff-Whats-the-difference-between-a-coder-programmer-and-an-engineer) and [here](https://www.quora.com/What-is-the-difference-between-developer-vs-programmer-vs-software-engineer-vs-computer-scientists).

I'm not gonna discuss about each terminologies one by one, rather I want to focus on 2 specific terms in this post, which is **Programming** and **Software Engineering**. 

One of the most direct answer in quora was like this:

- **Programming**: is simply someone who writes computer programs, can also be called a coder.
- **Software Engineering**: is the study and an application of engineering to the design, development, implementation and maintenance of software in a systematic method.

(It seems obvious the difference between them after reading that now)

Now let me continue by telling you a story. At the beginning of my career, Starqle got mostly outsourcing projects and we're hired to work under other company project manager, system analyst and business process analyst. My work was predominantly to code following exactly their design and specification. And most system analyst that I work with was usually not from a computer science background or haven't got much programming experience (let alone know or use Agile and Scrum methodologies).

One of the project that I working on at that time was not running very well. It was a big project for an enterprise with many modules and cross-department functionalities and I got the responsibility to work on 2 modules out of around 14. The problem was the design and specification that I (and other programmers) received was not much of a 'design' rather it was a template document hastily created without fully considering the actual needs of the customer and the implications of each design decisions. 

Naturally the project got delayed and time was mostly spent on redoing our works over and over again due to changes. And when it was actually going into production, issues pop up everywhere and we are still working as intense as it was before during the development (to fix bugs here and there)

I believe the story was not that uncommon in this industry, many people (esp. top management) still don't realize that merely *writes computer programs* are not enough when working on a larger scale project. It is especially important to also properly think about the **design** part of it. However, now comes another question: "Who is responsible for doing that?"

The answer is still system analyst and the system analyst should be a software engineer (In case you are confused, system analyst is the role name in a project, whilst software engineer is the profession name).

I'm a strong proponent of: "to design a program well, thou should be able to write a program well too". Thus, yes, I strongly believe becoming a software engineer is one of the obvious career path from a programmer. Software engineer is unlike architect in which you don't have to be a construction labor before becoming an architect. To become a good software engineer, you literally have to be a (good) code laborer. 

Continuing my story, after Starqle accumulate enough experience while building our own set of libraries and tools, I saw my role shifted more and more from programmer into software engineer. I spent more time reviewing code from my fellow teammates and removing lots of codes before merging (yes, I believe I remove more line of codes rather than adding some new ones). I also tinker a lot with pen and paper and discuss (or quarrel) with my teammate before writing any line of codes.

We also integrate Agile and Scrum practices more, which means less ceremony and more actual work done. I know this is a strong opinion here and that is because I still encounter clients (until recently! this story was a long time ago) whom still require us to submit lot of (useless) documents.

So back to software engineering, what is exactly the **role of software engineer** then? Was it to delete codes, sketch with pen & paper and quarrel with teammates? (sounds fun for me!)

To answer the above question, I really like to quote Fred Brooks from his evergreen book on software engineering

> “The programmer, like the poet, works only slightly removed from pure thought-stuff. He builds his **castles in the air**, from air, creating by exertion of the imagination.  
Few media of creation are so flexible, so easy to polish and rework, so readily capable of realizing grand conceptual structures”  

> Fred Brooks - The Mythical Man-Month

If software development is actually the process of building a castle in the air, where every components of it are purely stored inside imagination of each people in the project then software engineer role is to make sure that the components inside everyone head could actually be integrated to build the complete castle. They should also make sure that the castle is suited for the user requirements.

And how they achieve that, you ask? This is a lifelong-learning topic for me myself and right now I'm settled with deleting codes, sketch with pen & paper and quarrel with teammates.

Joking aside, Agile & Scrum currently win it for me as it strikes the balance for the volume of documentation generated (yes, documentation is still important in correct dosage and form). We should also cannot underestimate the power of pen & paper when explaining things to other people (helping them create components for the castle). I learn some techniques from [this](https://www.amazon.com/Back-Napkin-Solving-Problems-Pictures/dp/1591841992) book. One more thing that helps a lot is **testing**, especially automated testing and testing with proper test methodology.

I also expect newer and shinier techniques and frameworks for software engineering to emerge albeit it would be on a much much slower rate than the rate of new javascript frameworks & tools appearance (sorry javascript friends!).

Discussing in details software engineering techniques are interesting, but for now let me excuse myself, because I have a castle to be built.
