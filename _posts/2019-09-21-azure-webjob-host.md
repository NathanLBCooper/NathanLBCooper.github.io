---
layout: post
title: "Azure WebJob Host"
date: 2019-09-21 12:00:00
excerpt: "I've created a library that provides shutdown detection for Azure WebJobs. It's both for continuous services and finite duration method calls. The Azure WebJob SDK can do that already, but this was created to be a 'lighter' dependancy and to be less opinionated on how WebJobs are written."
categories: [azure, my-projects]
comments: false
image:
  feature: project.png
---

I've created a <a href="https://github.com/NathanLBCooper/azure-webjob-host" target="_blank">libary that provides shutdown detection for Azure WebJobs</a>. It's both for continuous services and finite duration method calls. The Azure WebJob SDK can do that already, but this was created to be a 'lighter' dependancy and to be less opinionated on how WebJobs are written.

### The project

The way Azure notifies a WebJob process it's about to be stopped is by placing (creating) a file at a path defined by the environment variable WEBJOBS_SHUTDOWN_FILE. This means Azure is going to stop the process in 5 seconds time (although that time is configurable). If you want to be able to react, so that you can gracefully shutdown, you'd better listen for it.

This library uses the Microsoft.Extensions.Hosting pattern to provide shutdown notifications to a continuous service, provided by the user as a *Microsoft.Extensions.Hosting.IHostedService*.

If this sounds like something you'd like to use in your project, you can <a href="https://github.com/NathanLBCooper/azure-webjob-host" target="_blank">check it out on Github</a> and follow links to the nuget packages on nuget.org. As always, there's a lot more detail on the Readme.

### What did I learn?

- I learnt a lot about Azure Webjobs
- I learnt a lot about the Hosted Service pattern
- I learnt how to create a clean separation between application level code from framework level code. When it came time to use this library (which I consider framework code for webjobs), I had to organise it cleanly together with the application code of that webjob.
- I tried my hand at testing with doubles instead of mocks. And I tried using less granular unit tests, to enable easier refactoring and a more clear relationship between the tests and top-level desired behaviour.
