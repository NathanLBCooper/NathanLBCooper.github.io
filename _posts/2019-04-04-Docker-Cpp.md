---
layout: post
title: "Using Docker containers to create a reproducible C++ build environment"
date: 2019-04-04 12:00:00
excerpt: ""
categories: [c++, docker, my-github-projects]
comments: false
image:
  feature: dockercpp.png
---

Based on a series on devblogs.microsoft.com:

- [Using multi-stage containers for C++ development](https://devblogs.microsoft.com/cppblog/using-multi-stage-containers-for-c-development/)

- [Using VS Code for C++ development with containers](https://devblogs.microsoft.com/cppblog/using-vs-code-for-c-development-with-containers/)

----------

I wanted to create modern C++ application. I've got a scenario: I'd like the application to sit there, listen for rabbit-mq messages, and do some heavy, difficult, C++ style work when the messages do come in. It's a semi-realistic scenario (I say "semi" because I'd use .net core in real life).

But I didn't want to go through the stress of configuring my windows machine to build my C++ project. And I didn't want to tie myself into Visual Studio C++ tools either. So I thought I'd try dockerising my builds. Luckily, Microsoft had just published a guide I found really useful. It's a guide on how to make a multi-stage container build for C++. It's "multi-stage", because you don't want the container you deploy to be bloated full with your build tools; just the stuff to run it.

I also didn't want to go through the hassle of building other people's code, or pulling their header files into my project. Luckily, if you follow the blog series, they use vcpkg, which is kind of like an npm for C++. 

I ended up with quite nice [C++ application](https://github.com/NathanLBCooper/rabbitmq-cpp-example) that talked to the world via Rabbit-mq and a Restinio web server. It took a bunch of fiddling, but it works (...everywhere, it's great!) and it's got a Readme that should make building and running it pretty easy.

I'll might write more about the specifics of getting my project working, but in the meantime, I encourage you check out the linked articles if this sort of thing interests you.