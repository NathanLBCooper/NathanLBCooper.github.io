---
layout: post
title: "Your software should build, test and push artifacts on YOUR machine"
date: 2019-07-28 12:00:00
excerpt: "How to avoid getting tricked by tutorials and get your Continuous Integration code right the first time"
categories: [continuous-integration]
comments: false
image:
  feature: "worksonmyciserver.png"
---

**It works on my machine.**

So the saying goes. It's a common problem developers have when they fail to anticipate how code might run differently outside their local environment.

Strangely enough, I occasionally see the exact opposite problem when it comes to Continuous Integration (CI). I've seen deployment scripts buried in CI configuration, and even the odd stackoverflow question on how to run a CI tool locally so the questioner can debug their deployment code. It's a situation that is typically too painful to last very long, but even so, in the interests of software developer public safety I'd like to issue a warning:

**Don't be tricked by tutorials.**

A developers is someone who reads a lot of tutorials. We've all got a lot of stuff to do and there are a lot of tools we've got to use to do it. But I think sometimes we forgot that tutorials are just tutorials, not a best practice example. This is as true with CI as anything else. The right way to quickly demonstrate concepts to a newcomers is rarely identical to the right way to do it for a real application. Ie, just because the AppVeyor example code inlines all the build code into their yaml file in order to get you started, doesn't mean you should commit that sort of thing to a real project.

The best place to develop code is your local machine. It's the quickest, easiest place, with the tightest feedback loop. That's after all why the phrase "It works on my machine" exists, because that's where we all start.

So if you haven't already, extract your build, testing and deployment scripts from your CI framework. And be careful to treat tutorials only as what they actual are, a learning tool.

