---
layout: post
title: "How to create responsive iframe content"
date: 2017-09-21 12:00:00
excerpt: "If your site is going to embedded in another one using an iframe, here's how to make that responsive and easy"
categories: [front-end]
comments: false
---

Iframes are not usually the posterboy of a quality web experience. But they are useful, and part of my dayjob used to be making a site that could be iframed into other organisation's websites.

A lot of getting iframes right is to do with good design, communication, documentation etc. But let's talk about two things you can do in your code right now to improve any iframed application you might be distributing.


##### Support resizing through the iframe

Go and use David J. Bradshaw's [iframe-resizer](https://github.com/davidjbradshaw/iframe-resizer) code. This is a fantastic bit of code, and the readme is easy to follow.

Be sure to turn the scroll bar off. A scrolling box within another page doesn't look or feel good. Taking the scroll bar away also encourages the host page builder to give your site the space it needs to breath.

##### Make embedding your site easy

Make it extremely difficult to screw up and make the default behaviour good. This is what it takes to embed the content I work on:

```javascript
<section id="nathan_thing">
        <noscript>Please enable JavaScript to view nathan's stuff</noscript>
</section>
<script>
    function nathan_config() {
        this.subroute = "YOUR_SUBROUTE";
    };
</script>
<script src="https://wheremyembedscriptis.com/assets/embed.min.js"></script>
```

There's very little here to get wrong. The person copying this code doesn't even have to know what an iframe is, they don't even have to know where my site is hosted, and I can change it at any time.

That's because my embed script looks like this:

```javascript
/**
 * Embeds the iframe into <section id="nathan_thing"></section>
 */
(function () {
    var config = new window.nathan_config();
    var baseUrl = "https://whenmysiteis.com/";

    var html = '<iframe id="nathans-id"' +
        ' src="' + baseUrl + config.subroute + '/someroute"' +
        ' width=100% allowtransparency="true" frameborder="0"' +
        ' tabindex="0" title="Nathans-thing"' +
        '></iframe>' +
        '<meta http-equiv="X-UA-Compatible" content="IE=edge">' +
        '<!--[if lte IE 8]>' +
        '<![endif]-->';

    document.getElementById("nathan_thing").innerHTML = html;

})();

```

Simple and easy. I removed a bunch of extra good stuff from this code for example purposes, so just in case you want the full thing <a href="{{ '/other/embedFullCode.txt' | prepend: site.baseurl }}">here it as a .txt file</a>.
