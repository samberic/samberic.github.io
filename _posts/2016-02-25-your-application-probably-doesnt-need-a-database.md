---
id: 136
title: 'Your Application Probably Doesn&#8217;t Need a Database'
date: 2016-02-25T23:20:42+08:00
author: Sam
layout: post
guid: http://samatkinson.com/?p=136
permalink: /your-application-probably-doesnt-need-a-database/
dsq_thread_id:
  - "4610438719"
categories:
  - Uncategorized
---
<i>This post originally appeared on <a href="https://dzone.com/articles/does-your-application-really-need-a-database">Dzone here.</a></i>

Ok, I’m being a little facetious here, but it’s certainly my default position. Developers seem to love to integrate databases into their applications without a thought for the requirements, despite the fact they end up moaning a lot about the database (and, if you’re in a big organization, the database team). It just seems to be the default position — “I am building an application so I will need a database and I will need Spring and I will need MQ and I will need &lt;Insert defaults here&gt;”. This is the wrong way of thinking.

People love habits. Once we’ve fallen into them it’s nearly impossible to escape, and countless books have been written to try and help. But as the old saying goes, the first step is admitting you have a problem. You’re addicted to Hibernate and Postgres. Sure everyone else is too, but that doesn’t make it right.

Let me say off the bat that databases aren’t inherently a bad thing. I don’t dislike them in the way I do Spring for example. Databases can be great and wonderful and exceptionally useful.  It’s just that it has become the default position, when in fact there are a lot of applications where a big, separate database isn’t necessarily needed.
<h2>So What’s the Alternative?</h2>
No Database!

RAM is a wonderful thing. It’s cheap. It’s incredibly fast. The more of your applications data you have in memory the faster it’s going to run. I can’t emphasise how much easier it is to store data in memory.

There’s no need to have a complex object mapping layer. There’s no need to deal with transactions. You just have data in Objects and it’s so easy to work with.

<strong>Obviously this won’t work for every application</strong>, but it will work for more than you think. The criteria:
<ul>
	<li>Stateless service.</li>
	<li>You must be able to bootstrap any data from other sources at startup.</li>
	<li>Assuming you aren’t able to have a big cluster of these apps for failover purpose, you need to be dealing with information small enough that the bootstrap process is quick.</li>
</ul>
This obviously works well for aggregating services that sit atop other data sources, a surprisingly common workflow which is going to become more common with the increasing popularity of microservices.
<h2>Phone a Friend</h2>
If you’re dealing with a fairly large amount of data and it requires processing in your application, you can quickly get to a point where start-up times are huge if bootstrapping from external sources. In a resource constrained environment, this can be a big problem.

It may also be impossible to bootstrap the data from downstream all the time. In finance for example a lot of data is generated for “Start of Day (SOD)” from downstreams, but offer no intraday sources. This means that if the application goes down intraday there would be no way to restart it without some sort of data store.

But still, put down the database to start with. In this system, we have a dump of data at the beginning of the day, and we then receive updates throughout the day. This is still stateless; each instance of the application will be identical as there is no state that is unique to each node.

If the application goes down we need a source of the updates we’ve received since start of day then you need to phone a friend.

Unless there’s a catastrophic failure, as long as you have a single node up it can provide the missing data to it’s peers. The mechanism is up to you; HTTP, MQ, file dump, whatever floats your boat. It’s just a way to bootstrap a new node.  Everything's still kept in memory.

There is obviously the risk that all nodes die and you can't get your state from anywhere else.  If this could be a problem in your system, why not try local files?
<h2>Files</h2>
Files are awesome. They’re well understood, can be human readable, have a great set of APIs around them and are fast. It is relatively simple to write an application's state out to file, particularly if you’re using an event sourced system — you don't need to edit "records", you just have a timeline of events which you can replay in.

Files are also a great way to speed up the recovery process; instead of sending a huge dataset over the wire from a peer, you can bootstrap from local files up to the point of failure, then just request updates from your peers for after that point.

This really does work in production. One of the systems I worked on was designed for speed and used an event log file for data recovery. It was super fast and quick to recover, and we never had to interact with the database team which was a huge bonus.
<h2>SQLite</h2>
Perhaps you’re really unwilling to give up on SQL. Maybe you don’t want to have to write and maintain your own serialization (I’ll admit, it can be a burden if not done well). Don’t rush back to your centralized DB! SQLite is a brilliant, super lightweight SQL implementation which stores your database locally in flat files. It’s super fast, it's in very active development, and has a whole heap of benefits (read more <a href="http://charlesleifer.com/blog/five-reasons-you-should-use-sqlite-in-2016/" target="_blank" rel="nofollow">here</a>). It doesn’t even run as a separate process; just include the dependency in your project and start using it.  You get to have all the semantics of a database, but without dealing with anything remote.
<h2>None of These Apply to My System!</h2>
I’ll admit that not every system can get away with a local file for storage. Some genuinely do need global access and consistency, in which case I encourage you to go crazy. But, if you’re designing a new system (or you’re struggling with your current one), don’t instantly jump to Postgres or MySQL. You could save yourself a lot of time and pain, as well as speeding up your application, by designing your system to store data in memory or on flat file.