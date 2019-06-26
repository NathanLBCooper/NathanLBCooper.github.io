---
layout: post
title: "Here are some useful aspnet core model binding packages"
date: 2019-06-25 12:00:00
excerpt: "I've written a model binder and a value provider for asp.net core!!! They do useful stuff you may need.
If you're interested in writing shorter, clearer arrays in your query strings, or you're interested in configuring alternative names for your parameters: Read this."
categories: [asp.net, my-github-projects]
comments: false
image:
  feature: "https://i.imgur.com/r1YOTX9.png"
---

I've written a <a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">model binder and a value provider</a> for asp.net core!!!

___

Okay, context.

I'm having a problem with query string length in my `GET` requests. They have arrays. Like this:

```
public class SomeRequest
{
	public int[] SomeNumbers{ get; set; }
}
```

And this is leading to some really really long query strings

`api/controller/action?SomeNumbers=1&SomeNumbers=2&SomeNumbers=3&SomeNumbers=4&SomeNumbers=5&SomeNumbers=6&SomeNumbers=7&SomeNumbers=8&SomeNumbers=9&SomeNumbers=10`

There are often limits to how long a query string can get before stuff breaks. And anyway, these are clumsy and ugly.

Before giving up and switching to `POST` (ie use bodies instead of query strings), I though I'd do some things to cut down length. Here's what my **Delimiting Query String Value Provider** does:

`api/controller/action?SomeNumbers=1,2,3,4,5,6,7,8,9,10`

And here's what adding my **AliasModelBinder** to the mix does:

`api/controller/action?n=1,2,3,4,5,6,7,8,9,10`

Maximal shortness accomplished.

If you're interested in doing the same thing with your query strings in asp.net core, or have another (tbh probably better) use for the ability to rename properties in your query strings, you can check out my  **<a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">ASP.NET Core model binding repository</a>**, and follow links to the **nuget packages on nuget.org.**
