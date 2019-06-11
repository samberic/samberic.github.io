---
id: 25
title: Dipping my toes in Node.js
date: 2014-03-23T10:49:19+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=25
permalink: /dipping-my-toes-in-node-js/
videourl:
  - ""
dsq_thread_id:
  - "2778003485"
categories:
  - geek
---
<p>Having played around a little with Node before in hack projects at work, I wanted to have a proper dive and get my hands a bit dirtier. As a result, I'm midway through another increment of http://lastband.in, this time with a node backend. &nbsp;Here's some things I've discovered so far</p>
<p></p>
<h2>Use Express 3.0 and persist with it</h2>
<p></p>
<p>I started out wanting to do a slightly different project connecting facebook to some other bits and pieces. &nbsp;When you sign up with Facebook it offers to kick you off with a basic project in Heroku, which is a very nice feature; If you don't know <a href="http://www.heroku.com/" target="_blank">Heroku</a>, it's a great place to deploy stuff for free when starting out and offers a lot of nice tools.</p>
<p>The basic app uses <a href="http://expressjs.com/" target="_blank">express</a>&nbsp;as it's HTTP mechanism. If you google there's a few "roll your own" solutions, then express and <a href="http://www.senchalabs.org/connect/" target="_blank">connect</a>.</p>
<p>I wanted to use handlebars as my templating library which should be easy enough, but I instantly ran against a ton of issues, mostly because I was ignorant to the ways of express; When Express went from Express 2 to 3 a lot changed, and it seems a lot of the internet hasn't caught up yet. All of the mixed documentation got me in a faff so I switched over to use connect. DO NOT DO THIS! It's very very difficult to google, there's very little documentation on the main site and the internet has very little to offer.</p>
<p>On the other hand, express is as close to a standard as you can get in node land, and once you wrap your head around it and find some reliable articles, it's actually quite usable!&nbsp;</p>
<p></p>
<h2>How to use Express 3.0 with Handlebars</h2>
<p></p>
<p>I was going to post the server up here but instead I've whacked it up on my github at <a href="https://github.com/samberic/express-handlesbars-starter">https://github.com/samberic/express-handlesbars-starter.</a>&nbsp;</p>
<p></p>
<h2>Node.js is still very immature</h2>
<p></p>
<p>Whilst node is undeniably awesome, very easy to use, and just a lot better for quick projects and fast progress, it's still at the very early stages. &nbsp;A number of the projects I tried using in npm just wouldn't work at all. Want to<a href="http://blog.nodejs.org/2012/04/25/profiling-node-js/" target="_blank"> profile your app</a>&nbsp;on a mac? Not a chance. &nbsp;All very frustrating, and would certainly be a blocker from using it in the enterprise.</p>
<p></p>
<h2>You're not in the browser any more</h2>
<p></p>
<p>For lastband, it worked in 3 tiers; &nbsp;first, find the band on last.fm, render a handlebar template. Then, find artists with that name on bandcamp, render a handlebar template. Then, for each artist, get their discography, and render a handlebar template, then add them all together with a bit of jquery.</p>
<p><span>This was the easiest way to do it in the browser (I thought at the time), as I was dealing with 3 different API calls. &nbsp;When porting to node I spent ages looking at JQuery replacement libraries (most of which are based on jsdom and are terrible/won't compile; If you're going down this route I recommend </span><a href="https://github.com/MatthewMueller/cheerio">Cheerio</a><span>) at building the html up in this incremental fashion.</span></p>
<p>I got this working, but it was slow, the code was fugly, and it just didn't feel right. &nbsp;As a result, I switched it up; after playing with JS for a bit I realised I could just append each API JSON result to the previous one, resulting in one massive JSON doc; Combine that with 1 single handlebars template, and voila! Now my template is clean and very obvious what's going on, and my code is to. &nbsp;</p>