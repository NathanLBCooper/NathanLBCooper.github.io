---
layout: post
title: "Your software should build, test and push artifacts on YOUR machine"
date: 2019-07-28 12:00:00
categories: [continuous-integration]
comments: false
---

**It works on my machine.**

So the saying goes. We wrote something that worked locally, but for whatever reason it doesn't work in the wild. Maybe it's a configuration problem, a missing dependency or maybe even   performance related. Either way, it doesn't work. The developer should have checked and they should have automated those checks.

But why did the developer start with the machine? ... Because it's the quickest and easiest way to develop code. It's the shortest feedback loop by a long way. Obviously, this is how we all work.

But, **why do we forgot this when we write our CI code?**

<br/><br/><img src="{{ '/img/worksonmyciserver.png' | prepend: site.baseurl }}" style="max-height: 20em;"><br/><br/>

I've seen too much of the code we use to build and deploy our applications buried in appveyor *.yaml* files, or worse, completely absent from source control and written in some Teamcity textbox. I'm not a fan of this. If I want to change how an application builds I don't want to have to go hunting on a CI server. 'Teamcity-debugging' isn't productive or fun, I just want something I just run and develop in quickest and easiest place: *on my machine*.

Here are some things I do before even thinking about CI: I write a build/test script, I decide where my artifacts go, and I write a deploy script. If I'm writing something that makes a nuget package, I can send version 0.0.1 before I even touch a CI server. Working locally is just the quickest way to resolve any problems.

Here's an example of an Appveyor file I wrote recently:

	...
	build_script:
		- ps: .\build.ps1 -build_number $env:APPVEYOR_BUILD_NUMBER -branch $env:APPVEYOR_REPO_BRANCH
	artifacts:
		- path: .\build\nupkgs\*
	deploy_script:
		- ps: .\deploy.ps1 -nuget_source $env:nuget_source -nuget_api_key $env:nuget_api_key
	skip_commits:
  	files:
    	- '**/*.md'
    ...

I let the CI do all the things it's good at doing automatically, like picking when to run and when not to. **But the things I have to write and maintain, my `build_script` and my `deploy_script` are trivial to get working locally**. I've made them parameterized powershell scripts on the highest possible level, and I won't need to do a whole lot of typing to get things working on my machine.

Fixing and maintaining them in future will be much easier. I'm also much less locked into appveyor.

Let's start doing this with the CI code we write from now on.