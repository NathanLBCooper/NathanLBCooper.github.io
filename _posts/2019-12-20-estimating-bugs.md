---
layout: post
title: "Is Pivotal Tracker right? Should bugs go un-estimated?"
date: 2019-12-20 12:01:00
excerpt: "If you've ever used Pivotal Tracker you'll have noticed bug estimation is off by default. Why is that? And is it the right thing to do: should bugs be estimated or not?"
categories: [process]
comments: false
image:
  feature: process.jpg
---

**Should we estimate bugs?**

Here's what Pivotal have to say:

> Feature stories are estimated because they contribute to business value. Bugs and chores are considered part of normal software product overhead—they emerge over time, and are continual overhead, an ongoing cost of doing business. Tracker’s automatic velocity calculation accounts for this as a built-in cost, and estimates how much business-valued work can be completed each iteration. This lets you focus your planning on business value, risk, and priorities. Therefore, bugs and chores in Tracker are not normally estimated.

I don't think they mean that fixing bugs doesn't literally "contribute to business value" (because that would be an insane thing to say), but that it doesn't progress the team further through a feature based road-map. Let's call this "feature-velocity", and we can say "fixing bugs doesn't add to feature velocity". So, according to Pivotal, we aren't going to collect capacity data for all the work we do, we're only going to record estimates for the things that are part of our feature-velocity.

In order to figure out if that's a sensible thing to do, let's go back and look at why we collected it in the first place. This is why Pivotal thinks we should estimate:

> "Tracker uses velocity to plan future iterations in the Backlog with a reasonable amount of work, based on the project team’s historical workload"

This is a sensible objective. But unfortunately it's a capacity based decision, not a feature-velocity based one. It doesn't matter whether that work takes you further down your road-map. Work is work and work takes time. There is such a thing background work and overhead, but the work we can prioritize and make decisions about when planning iterations is not overhead. It's plannable work and it's equal as far as capacity planning is concerned, regardless of its contribution towards feature-velocity.

It's not that feature-velocity isn't valuable, it can be. Self-evidently, it tells us how much time we get to progress through a road-map. Which, as long as we realize that time spend doing valuable things isn't an exact proxy for value, is a metric we can learn from. It's just not a good idea to throw away our capacity data, and then try to make capacity decisions based on our feature-velocity.

*So, should you keep the Pivotal default and not estimate bugs?*
**No. No you shouldn't. Estimate your bugs**

.. Unless...

If you're a team that don't plan and prioritize bugs, but instead always fixes them as soon as they are found, my capacity arguments don't apply. In this case you can try out treating bug fixing an overhead task. How good of an idea that is can be the subject of another post.