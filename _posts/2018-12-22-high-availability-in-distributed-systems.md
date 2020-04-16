---
layout: post
title: "Keeping availability high in distributed systems"
date: 2018-12-22 12:00:00
excerpt: "How much like the Titanic is your microservice architecture? How tightly bound is the success of your entire systems to the stablity of it's individual components? Is there a risk that your new microservices might be destroying your availability?"
categories: [fault-tolerance]
comments: false
---


On April 10, 1912, the R.M.S. Titanic set sail from Southampton, England, to New York. The Titanic was one of the biggest and advanced state of the art ocean liners of its day. Equipped with a host of new safety technologies, it was said to be "unsinkable". But four days into the ship's maiden voyage, she sank after colliding with an iceberg, tragically taking the lives of around 1500 passengers and crew.

The Titanic had 16 watertight compartments. The ship could stay afloat with up to four of these compartments flooded. The blow to her starboard side opened five compartments to the sea, and she sank two hours and forty minutes later.

<img src="{{ '/img/titanic.jpg' | prepend: site.baseurl }}" alt="Titanic Sinks" style="width: 60%; height: auto;">

Here's the twist. How many water tight compartments does your software system have? How many of your shiny new micro-services could non-iceberg-related server incidents take out before your entire system fails? Sure. Early 20th century ship-makers had serious problems with their disaster recovery/mitigation planning. But as far as building our ships, or our applications, might they have been better masters of their craft than we are of ours?

In the software field, we're very good at thinking about single-process non-distributed design. The patterns and abstractions we reason with are perfectly fitted to our understanding that "code call other code => other code works". In distributed systems this looks more like "code calls other code => other codes works... maybe...maybe later... in a different order". This has huge implication for how we design software. One of these implications is that we're going to have thinking a lot more about dealing with resiliency in our application code.

By "resilience", I mean "the ability to handle unexpected situations, either without the user noticing (best case) or with graceful degradation of service (worst case)".

Our understanding of design has to change. When we break up our code, we will have to break it up into functional units that can do stuff on their own. We have to use bulkheads. Just like the Titanic, we still need to be able to float if we lose a few of them to the sea.

Let's look at a simple example. 

> I'm building a "Trivia" micro-service that tests our users' knowledge. It will greet them by name and ask them some fun questions.
> We already have another "User" service that knows who they are. I'm going to either 1) assume that the "User" service will never fail or 2) decide that I can't quiz a user I can't identify.
> In either case the "Trivia" is now doomed to fail when the "User" service does.
> I continue to build the rest of my software out of micro-services with reasonably coherent and separate purposes, but without really thinking about isolation.

Now it doesn't take an "iceberg" breaking through six bulkheads to sink my "ship", one will do. It's not very resilient. In fact, the more services we make the more scary this becomes.

Let's talk about availability.

Availability is multiplicative. If you need everything to work, you need to multiply your availabilities. Let's say our servers are up 99.9% of the time. Your "grandfather's" software setup might require a few machines here and there, but which all have to work. Say 3. That's 99.7% uptime, which isn't bad. Maybe operations staff can solve this by buying better gear, so there's no need to involve the application developers. But let's up that to 50 machines. 99.9% times 99.9% fifty times is 95.1%. Our 3-nines system is now down 5% of the time. Increase the machines again to 100 and it's down 10% of the time. This is no longer an Ops problem. We have to ensure isolation properly in our application design if we want available systems.

When we design our services we must ask ourselves "what can we do if the other services aren't there". If the only answer we can see to that question is "fail", then the design is wrong. Let's get this right, and then someday your system will be as robust to partial failure as the Titanic was.

We'll talk about lifeboats later.

------------

PS

I recently was lucky enough to attend a seminar by Uwe Friedrichsen, which explored a lot of the issues around designing distributed systems. He touched upon isolation and designing for failure, which is what inspired this article. I'm definitely going to be checking out more of his stuff. And if you're interested in this subject at all, I'd recommend watching his video on [The 7 Quests of Resilient Software Design](https://www.youtube.com/watch?v=qWo57fWvycU)

*Image Credit: Untergang der Titanic" by Willy Stöwer, 1912*