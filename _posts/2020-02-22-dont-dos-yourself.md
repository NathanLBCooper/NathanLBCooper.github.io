---
layout: post
title: "Exponential Back-off: Don't DOS yourself!"
date: 2020-02-22 12:01:00
excerpt: "Make sure to back off requests when your upstream infrastructure is struggling"
categories: [programming]
comments: false
image:
  feature: code.png
---



Something went down. Hard. In production. It's one of those days.

External traffic was spiking very high, and things got bad. But the problem seemed to continue and continue even after the initial spike of external traffic had passed. Arghh. Bad. Why?

Because another downstream service in our system had a simple but absolutely terrible response to its upstream requests failing. Silently waiting to bite me at the worst possible moment. Under normal conditions, it would run fine. But if something upstream of it failed, it would retry and retry and retry... Multiplying the existing traffic, adding more and more load to the already struggling service, making the outage longer and longer.

I spend enough time trying to stop outsiders doing bad things to our system, I don't need it trying to break itself.

Remember to use **Exponential Back-off** when retrying requests.

What does it mean to use Exponential Back-off? It means that every time you retry an operation, you wait for a longer and longer (exponentially increasing) time. This means one-off failures are retried quickly, but if somethings genuinely wrong the retry traffic doesn't snowball into a huge traffic monster. And that's the change I'm going to be making to the downstream service today.
