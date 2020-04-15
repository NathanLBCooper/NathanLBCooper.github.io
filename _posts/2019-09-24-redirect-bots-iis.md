---
layout: post
title: "Redirect Crawlers away from your IIS hosted webpage"
date: 2019-09-24 12:01:00
excerpt: "Then a link to a website is posted somewhere like Facebook, Twitter or Slack, a crawler will follow that link. That's how Facebook, for example, grab titles, descriptions and images from their links. If you want them to stop following your links, or show them alternative content, here's how."
categories: [how-to, programming, iis]
comments: false
---

When a link to a website is posted somewhere like Facebook, Twitter or Slack, a crawler will follow that link. That's how Facebook, for example, grab titles, descriptions and images from their links.

There might be links you do not want these bots to follow. Or maybe links where you just want to show these bots different content than you would show the humans. Luckily, you can redirect just the bots using IIS.

First, you'll need to install the module. Here's how to do it on the command line using DISM:

`Dism /Online /Enable-Feature /FeatureName:IIS-HttpRedirect`

Then, in your *web.config* , under *system.webServer*, just add a re-write rule:

    <rewrite>
        <rules>
        <rule name="Redirect Crawlers">
            <match url="(.*)" />
    This conversation was marked as resolved by infl3x
            <conditions logicalGrouping="MatchAll">
                <add input="{REQUEST_URI}" negate="true" pattern="robots.txt$" ignoreCase="true" />
                <add input="{HTTP_USER_AGENT}" pattern="googlebot|Google \(\+https|adsbot-google|mediapartners-google|googleimageproxy|facebot|Twitterbot|facebookexternalhit|ia_archiver|baiduspider|sogou|360Spider|mj12bot|bingbot|simplepie\/|yahoo! slurp|duckduckbot|yandex\.ru\/bots|yandex\.net\/bots|yandex\.com\/bots|Exabot|Slackbot-LinkExpanding|MSOffice|Microsoft Outlook|SkypeUriPreview" />
            </conditions>
            <action type="CustomResponse" statusCode="204"/>
            </rule>
        </rules>
    </rewrite>

Replace the CustomResponse with something more elaborate (if you want), and change the match url to something more specific (also if you want) and you're done. Just be sure not to add this re-write rule before installing the module, which incidentally is disabled by default, otherwise your site will break.

This is easy to test in a browser. Here's how to do it in Chrome: Just look in your Devtools for "Network Conditions". Modify your user agent one used by a known bot, eg *AdsBot-Google (+http://www.google.com/adsbot.html)*. Then try to navigate to your endpoint and verify that you're redirected.
