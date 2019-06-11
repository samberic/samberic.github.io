---
id: 43
title: Why I hate Spring
date: 2014-03-23T14:42:04+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=43
permalink: /why-i-hate-spring/
videourl:
  - ""
dsq_thread_id:
  - "2655782078"
categories:
  - geek
---
When I started my career I really fell in love with <a href="http://projects.spring.io/spring-framework/">Spring</a>. I went long on it. I threw it into all of my projects. I even managed to chuck a bunch of <a href="http://projects.spring.io/spring-integration/">Spring Integration</a> in there for good measure. I was like an XML king.  I was building was a custom RPC layer based over JMS, protobufs and <a href="http://kaazing.com/">Kaazing</a> to be used across our department and further around the bank. "It's so configurable" I would say. "It's just a few XML files, it's really flexible" I would say. I was pretty pleased with myself.

The thing is, there were some people around me that tended to disagree. They were having issues getting this wired together how they wanted, they didn't know which spring XML files they needed where. There were issues with spring versions and getting the right versions together (I'd also gone long on modularisation; there was about 5 or 6 different modules at different version numbers with no obvious way, other than an email from me, to know which to use). I didn't notice these smells; I just thought it must need more documentation or that the people using it were being stupid. This is a pattern that repeats itself too; for one of the most disliked and difficult to use frameworks internally currently cries for help are often met with "It's one file and some parameters, it's not that hard" whilst everyone else wastes days trying to get the magic combinations of files and parameters to make something happen.

I'm still in the same organisation, and in my new role I'm a consumer of my old framework. This dogfooding has now led me to hate 2009/10 era Sam for several reasons, but mostly for spring. Spring is evil on a good day, but when it's included as part of a consumable library or API it becomes next level, like a love child of Hitler and the devil. Don't let Spring leak out of your APIs.

There are a number of reasons why Spring sucks which I felt the need to document, as nothing's showing up on google as a concise argument against it.
<ul>
	<li><strong>Configuration in XML :</strong> I'd like to think that as a profession we'd moved beyond XML. It's incredibly verbose but that's just a minor starting gripe. Much more importantly, I don't want to program in XML. The wiring of all of your classes together is a hugely important part of your application. You're a java developer, not an XML developer. One of the beauties of java as a language is <strong>compile time safety</strong>. In my springless applications I can hit compile and have a 100% certainty everything's built, plugged in and ready to work. In applications I work on with Spring, you hit run, wait for 30-60 seconds whilst it initialises beans, before falling over. In the modern world we live in this is insane, particularly when you multiply that up over a number of integration tests where you need to spin the container up and down.   There's also a special place against the wall for the "It means I can change my implementation without recompiling!".  No one does this.  Ever.</li>
	<li><strong>Magic : </strong>At this point the usual come back is "you can do it all via annotations now! No more xml!".   Whilst not programming in XML is swell and all, annotations are still magic.  Until you run your app you've no idea if it's wired up correctly.  Even then you don't know it's wired up correctly, only that it's wired up.  I don't like magic.</li>
	<li><strong>Importing other Spring files : </strong>This is currently the item that causes me the most rage.  I've discovered there's a tendency to break Spring files down into smaller spring files, and then scatter them across modules.  I've just spent 2 weeks of crawling through jars trying to find the right combination/order/version of spring files to make something actually run.  Spring files in jars is a bad, bad idea.  Terrible.  Every time you spread dependent spring files across jars a child dies.</li>
	<li><strong>Complexity</strong> : When interviewing candidates the most common answer to "any pitfalls to Spring?" is that it has a steep learning curve. Whether or not that's true is another discussion for another day, but I wanted to highlight the fact that Spring is now so complex that it has it's own framework, <a href="http://www.infoq.com/articles/microframeworks1-spring-boot">Spring Boot.</a> A framework for a framework. We are in Framework Inception, a film about Leonardo Di Caprio trying to find his long lost java code by going deeper and deeper through layers of XML and annotations before eventually giving up on life.</li>
</ul>
The thing is, I'm sure it's possible<em> in theory </em>to use Spring well in an application.  I've just yet to see it happen, and this is the problem.  And for me all of the "benefits" it offers are perfectly possible without it.  When asking about Spring as a part of our interviewing process and the standard answer is "it means you have clean code, separation of concerns, and it's really good for testing".  All things I'm huge fans of (in particular the testing part) but the simple fact is these are not outcomes of using Spring, but outcomes of programming well.  Perhaps Spring is a good crutch for new developers to use to be introduced to the ideas of dependency injection and mocking and testing, but the simple fact is they're orthogonal.  If you TDD your code you'll find no getters and setters, only constructor injection which you can mock for tests, and then when you're putting your application together, just use the often forgotten about construct, the "new" keyword.  We often build a class called "ApplicationContext" which is in control of wiring everything together.  It's clean, everything's testable, I have compile time safety and my tests run darn quickly.

&nbsp;

&nbsp;