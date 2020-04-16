---
layout: post
title: "Engineering Chaos"
date: 2015-09-29 12:00:00
excerpt: "This is a indispensable piece of advice: “The best way to avoid failure is to fail constantly”"
categories: [fault-tolerance]
comments: false
---

The [Netflix Tech Blog](https://medium.com/netflix-techblog/5-lessons-weve-learned-using-aws-1f2a28588e4c) has a number of recommendations for anyone moving to the cloud.

Possibly the most indispensable piece of advice the blog gives is this:  **“The best way to avoid failure is to fail constantly”**.


> We’ve sometimes referred to the Netflix software architecture in AWS (Amazon Web Service) as our Rambo Architecture. Each system has to be able to succeed, no matter what, even all on its own. We’re designing each distributed system to expect and tolerate failure from other systems on which it depends. […]
>
> One of the first systems our engineers built in AWS is called the Chaos Monkey. The Chaos Monkey’s job is to randomly kill instances and services within our architecture. If we aren’t constantly testing our ability to succeed despite failure, then it isn’t likely to work when it matters most – in the event of an unexpected outage.

<img src="{{ '/img/chaos-monkey.jpeg' | prepend: site.baseurl }}" alt="Chaos monkey" style="width: 60%; height: auto;">

This advice might sound crazy. Why would you actively attack your own deployed services? Because... regardless of what you do... individual components of your solution will fail. Even the most robust equipment eventually fails and if a system cannot both recover from small failures and gracefully degrade from larger ones, then that system is only as strong as its weakest link.

Netflix compares this to getting a flat tyre on the motorway. The guy who’s been practicing every weekend by popping and replacing his tyres on his drive is going to be much better prepared than the guy who hasn’t looked at his spare in 6 months and doesn’t even know if it’s inflated.

With this philosophy Netflix has built something they call a 'Simian Army'. First, to ensure production instances failing did not affect customers, they created the 'Chaos Monkey'. This is a tool which randomly disables instances. Other simians followed. The 'Latency Monkey' was created to introduce artificial delays and the 'Chaos Gorilla' to simulate Amazon availability zone outages.

The company began constantly testing their resilience on their real production systems. This wasn't some test that could be skipped or ignored, or that was real as testing gets. It was also a clear statement that any code that got deployed had better be resilient. They were creating a development culture around resiliency and preparing the best they could for the real deal.


The ‘tyres burst’ when AWS suffered significant networking issues across the Americas on Christmas Eve 2012. Incredibly, the Netflix web site remained up, at a time when many popular websites crashed. While some streaming services for users in the Americas were degraded, nothing broke. Their commitment to recovery from failure had paid off.

We can all learn from Netflix. A lot of the lessons are cultural. Netflix had hired smart people and trusted them to make decisions and take smart risks. They had a technical culture where people thought carefully about resiliency and had the space to spend time working on it. That's a hard thing to build and a very valuable asset for a company. But there are easier technical things we can start doing as well. Let's start talking seriously about resiliency. Let's start testing our code for it. And we adjust our approach to coding to not just think "how will this work", but also "how will this fail". 

*Image credit [Netfix Tech Blog](https://medium.com/netflix-techblog)*