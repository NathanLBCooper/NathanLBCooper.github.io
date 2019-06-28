---
layout: post
title: "The query strings in my asp.net core project used to be terrible. Adding this library changed everything"
date: 2019-06-25 12:00:00
excerpt: "If you use asp.net core, and are interested in writing shorter, clearer arrays in your query strings, or you're interested in configuring alternative names for your parameters: Read this."
categories: [asp.net, my-github-projects]
comments: false
image:
  feature: "https://i.imgur.com/r1YOTX9.png"
---

I've written a <a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">model binder and a value provider</a> for asp.net core!!!

___

Okay, context.

I'm had a problem with query string length in my `GET` requests. They have arrays. Like this:

```
public class SomeRequest
{
	public int[] SomeNumbers{ get; set; }
}
```

And this led to some really really long query strings

`api/controller/action?SomeNumbers=1&SomeNumbers=2&SomeNumbers=3&SomeNumbers=4&SomeNumbers=5&SomeNumbers=6&SomeNumbers=7&SomeNumbers=8&SomeNumbers=9&SomeNumbers=10`

There are often limits to how long a query string can get before stuff breaks. And, besides, this query string is ugly.

Before giving up and switching to `POST` (so I can use bodies instead), I thought I'd do some things to cut down length. Here's what my **Delimiting Query String Value Provider** does:

`api/controller/action?SomeNumbers=1,2,3,4,5,6,7,8,9,10`

And here's what adding my **AliasModelBinder** to the mix does:

`api/controller/action?n=1,2,3,4,5,6,7,8,9,10`

Maximal shortness accomplished.

*Okay, maybe renaming `SomeNumbers` to `n` is overkill here, but I can think of a lot of other uses for being able to alias variables. Backward compatibility is one. Perhaps you can think of others.*


If you have similar problems with long query strings, or would otherwise like to use one of these tools on your project, you can check out my  **<a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">ASP.NET Core model binding repository</a>**, and follow links to the **nuget packages on nuget.org.**
