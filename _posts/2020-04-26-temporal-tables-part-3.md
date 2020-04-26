---
layout: post
title: "Temporal Tables Part 3: Querying"
date: 2020-04-26 12:01:00
excerpt: "How query the history saved in your temporal tables"
categories: [storage]
comments: false
---

Actually, I'm not going to write an article about how to query temporal tables. Clark Dave has already [written a fantastic tutorial](http://clarkdave.net/2015/02/historical-records-with-postgresql-and-temporal-tables-and-sql-2011/) that explains everything.

Clark's article will show you how to Backfill history, query the state of your system at any given time, query when changes happened, query the volume of changes on a given date. You'll also learn that easy querying of history relies on creating a view on top of your entity and history tables. We'll be using Clark's approach in future articles.

Go read that and then we'll continue with the series.
