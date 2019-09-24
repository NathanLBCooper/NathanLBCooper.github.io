---
layout: post
title: "PROJECT -- Alias Model Binder + Delimiting Query String Value Provider"
date: 2019-06-25 12:00:00
excerpt: "I've created two utilities for asp.net core: 1) a model binder for allowing alternative property names on request models. 2) a value provider allowing alternative collection query variable syntax"
categories: [my-projects, programming, asp.net-core]
comments: false
image:
  feature: project.png
---

I've written a <a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">model binder and a value provider</a> for asp.net core.

### The project

I had a problem with query string length in my `GET` requests. Array were being passed in the queries, like this:

```
public class SomeRequest
{
	public int[] SomeNumbers{ get; set; }
}
```

And this led to some really really long query strings

`api/controller/action?SomeNumbers=1&SomeNumbers=2&SomeNumbers=3&SomeNumbers=4&SomeNumbers=5&SomeNumbers=6&SomeNumbers=7&SomeNumbers=8&SomeNumbers=9&SomeNumbers=10`

Which isn't great. There are often limits to how long a query string can get before stuff breaks. And, besides, this query string is ugly.

One option is to just switch to `POST` and start using request bodies instead. But I thought I'd do some things first to cut down length.

First, here's what my **Delimiting Query String Value Provider** does. It allows for a less verbose style of array syntax in queries.

`api/controller/action?SomeNumbers=1,2,3,4,5,6,7,8,9,10`

Second, here's what my **AliasModelBinder** does. It allows use of alternative property names in queries, without actually having to rename the properties.

`api/controller/action?n=1,n=2,n=3,n=4,n=5,n=6,n=7,n=8,n=9,n=10`

And, of couse, if we're playing URL golf, here is the combination:

`api/controller/action?n=1,2,3,4,5,6,7,8,9,10`

If you have similar problems with long query strings, or would otherwise like to use one of these tools on your project, you can check out my  **<a href="https://github.com/NathanLBCooper/aspnet-core-model-binding" target="_blank">ASP.NET Core model binding repository</a>**, and follow links to the **nuget packages on nuget.org.** There's a lot more detail there for anyone who wants to use this code in their web projects.

-----------------

### What did I learn?

- I learnt a lot about asp.net core and it's pipeline between receiving requests and passing it to the controllers.
- This was also the first project I set up on appveyor.