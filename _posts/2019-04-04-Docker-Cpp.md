---
layout: post
title: "Using Docker containers to create a reproducible C++ build environment"
date: 2019-04-04 12:00:00
excerpt: "I quick overview of a really interesting series of Microsoft blog posts, and the application I built following them"
categories: [c++, docker, my-github-projects]
comments: false
image:
  feature: dockercpp.png
---

I've been reading a really interesting series on how to make a multi-stage build on docker for your C++ project. Here it is on devblogs.microsoft.com:

- [Using multi-stage containers for C++ development](https://devblogs.microsoft.com/cppblog/using-multi-stage-containers-for-c-development/)

- [Using VS Code for C++ development with containers](https://devblogs.microsoft.com/cppblog/using-vs-code-for-c-development-with-containers/)

I've followed along. [Here's my project](https://github.com/NathanLBCooper/rabbitmq-cpp-example). It's a sort of template for a modern C++ project. There's a Restinio web server and it's a rabbit-mq subscriber and publisher. It uses that multi-stage docker build (of course) and I use Vckpg as my library manager.

If I was to see C++ as part of a modern application architecture, this is where I'd see it going. If someone gave me a large chunk of c++ that did a very difficult and strenuous thing, I'd probably put it between two queues, add some monitoring and then forget about it. *(Or maybe I'd hunt down the person who wrote it and introduce them to .netcore, I don't know)*

I think what's really nice about this project, is that **it runs basically anywhere, without any sticky C++ specific configuration**. I guess that's the point of developing on containers, but I've developed C++ on Windows before, and not having to do all the bullsh*t feels great. It also feels great not being tied to Visual Studios and its C++ tools. I ended up using Visual Studio Code and debugging with it into a container, and it was a really smooth experience.

**It's a multi-stage build**. What that means is, that while you create large and "powerful" containers to build and debug your application, in the end you create and deploy a container much smaller with just the tools needed to run it.

**It uses Vckpg**. This is sort of like using a 'normal' package manager (npm, pip etc), but different in a lot of interesting ways.

Briefly speaking, they've sort of given up on the idea on having multiple versions of packages, and you basically need to just accept a snapshot in time for all the libraries you use. You pick a point of time on the vcpkg commit history, and then the versions they have recorded as "current" at the point in time are what you use. It's a really interesting approach. You won't run into a dependency hell situation (I hope), but it's also very inflexible... Or maybe it's not inflexible, because you can fork vcpkg and change that version snapshot in any way that please you... I sense I'll have to go into more depth into vckpg (and some alternatives) later.

Let's just say using any package manager is still a million times better than pasting in header files and building other people's code, and vcpkg does the job.

Overall, this blog series is really interesting. I found this to be a bit of crash course in both docker and C++, both of which I got a lot of value out of. Please do check it out if this sort of thing interests you.
