---
layout: post
title: "How to use Octopus substitute variables In JavaScript files"
date: 2017-10-23 12:00:00
categories: [octopus, deployment, how-to]
comments: false
---

Octopus package steps have a feature that allows you to [replace Octopus Variables in any file](https://octopus.com/docs/deploying-applications/substitute-variables-in-files). 

Great, but I use that in JavaScript it needs to work locally before any octopus magic happens. So I use this:

```javascript
var baseUrl = octoTemplate("#{portalUri}", "http://localhost:4200/");

....

function octoTemplate(value, fallback) {
    if (value.substr(0, 2) === "#{") {
        return fallback;
    }

    return value;
}

```
