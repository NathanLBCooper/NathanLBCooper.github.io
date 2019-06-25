---
layout: post
title: "Download these nuget packages and start using shortened property names aliases in your query strings"
date: 2019-06-25 12:00:00
excerpt: "I've written a model binder for asp.net core!!!"
categories: [asp.net, my-github-projects]
comments: false
image:
  feature: "https://camo.githubusercontent.com/42f92bd9f47279b25eb5ca030c71ade0cf8ab922/68747470733a2f2f692e696d6775722e636f6d2f797245726c53582e706e67"
---

I've written a <a href="https://github.com/NathanLBCooper/alias-model-binder" target="_blank">model binder</a> for asp.net core!!!

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

Before giving up and switching to `POST` (ie use bodies instead of query strings), I thought I'd see if I could alias the property names to cut down on length. I don't want to change the names in my models to a bunch of incoherent letters, but I do want to just use letters in my query strings sometimes. Like this:

`api/controller/action?n=1&n=2&n=3&n=4&n=5&n=6&n=7&n=8&n=9&n=10`

Turns out, for asp.net core, I needed to write my own model binder.

**So I wrote it and put it on Github and Nuget.org.**

Here it is: 

**<a href="https://github.com/NathanLBCooper/alias-model-binder" target="_blank">Alias Model Binder</a>**
