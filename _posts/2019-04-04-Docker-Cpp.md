---
layout: post
title: "PROJECT -- How I learnt to create reproducible C++ build environments with Docker"
date: 2019-04-04 12:00:00
excerpt: "I quick overview of a really interesting series of Microsoft blog posts, and the application I built following them"
categories: [my-projects, programming, C++, docker]
comments: false
image:
  feature: dockercpp.png
---

I've started reading the [Visual Studio C++ Team blog](https://devblogs.microsoft.com/cppblog/).

I'm not entirely sure why. I'm a .NET developer after all. One some level, I feel that C++ developers are the real-men of the programming world. I'm jealous... but not enough to actually work in C++. But back to the blog! It's suprisingly great. Who knew? In fact, reading Marc Goodner's articles inspired me to follow along and brush up on some long-neglected C++ skills.

I'm talking about his really interesting series on how to make a multi-stage build on docker for a C++ project. Here are the links:

- [Using multi-stage containers for C++ development](https://devblogs.microsoft.com/cppblog/using-multi-stage-containers-for-c-development/)

- [Using VS Code for C++ development with containers](https://devblogs.microsoft.com/cppblog/using-vs-code-for-c-development-with-containers/)

It's been an education for me. One of the worst things about C++, as far as I'm concerned, is deploying it. Wrap it in a docker container and, suddenly, that's not a problem. Wrap both the building and debugging in a container, and now one of the next worse things, setting up your environment, becomes just stupidly easy. I like to develop on windows, but not in Visual Studio. This is probably one of the hardest development environments to setup, so I'm really appreciate using docker so far.

Let's explain some of the concepts I encountered while reading these articles and while building my own docker c++ project.

**What do I mean by multi-stage build?** If you're using a compiled language, you generally need more tools to build it than you need to run it. So when you deploy your application, you don't want to deploy a container bloated with a bunch of now-useless build tools as well. You want to have a build container that produces another smaller run container. Plus, maybe you also want to debug? Sometimes that takes even more tools that running it. So you also might want a even bigger container for that.

**What's this Vckpg thing it's using?** This is sort of like using a 'normal' package manager (npm, pip etc), but different in a lot of interesting ways.

Briefly speaking, they've sort of given up on the idea on having multiple versions of packages, and you basically need to just accept a snapshot in time for all the libraries you use. You pick a point of time on the vcpkg commit history, and then the versions they have recorded as "current" at the point in time are what you use. It's a really interesting approach. You won't run into a dependency hell situation (I hope), but it's harder to make a flexible choice of what dependencies you take.

Let's just say using any package manager is still a million times better than pasting in header files and building other people's code, and vcpkg does the job.

**So, here's my project:** [This thing](https://github.com/NathanLBCooper/rabbitmq-cpp-example). It's a sort of template for a modern C++ project. There's a Restinio web server and it's a rabbit-mq subscriber and publisher. It uses that multi-stage docker build (of course) and I use Vckpg as my library manager.

It's a bit rough though. After all that effort I got bored writing the actual C++ code to go inside all these fancy containers. Sadly, after all this, I'm still not tempted by the language. I guess I won't be taking my place amongst the C++ real-men any time soon.

This whole exercise did make me think about how I'd integrate C++ into one of my projects. I definitely would try and stick it into one of these builds. I would add a C++ package manager if there wasn't one. I'd also put it between two rabbitmq queues if the C++ was  build for some kind of heavy asynchronous number crunching, like I imagined in my example. Or maybe I'd just hunt down these hyperthetical C++ developers and explain the magic of .NET Core to them. Who knows.

But, despite my lack of enthusiasm for actually writing C++, I'll keep following these blogs. They're interesting. They inspire me to dig into things I otherwise won't. Check them out. Maybe you'll enjoy them as well.
