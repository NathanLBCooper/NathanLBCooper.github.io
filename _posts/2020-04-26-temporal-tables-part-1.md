---
layout: post
title: "Temporal Tables Part 1: What is a temporal table?"
date: 2020-04-25 11:01:00
excerpt: "What is a temporal table and why would you use one?"
categories: [storage]
comments: false
---

Recently, I was looking for a convenient way to track changes in a data-set over time. I came across something called 'Temporal Tables'. They're something you can add to your SQL database that adds easy support for storing and querying the state of your entities at any point in time in the past. This was good for me, as I wanted to analyse that change, but it's good for use-cases like audit. 

In order to show you what a temporal table looks like, let's dream up an entity first. Here's table of what's in my fridge:

    FridgeItem Table:
    | Id  | Type  | State |
    | 1   | Milk  | Fresh |
    | 2   | Juice | Fresh |
    | 3   | Beer  | Fresh |

This is the state of thing now. We're going to make a few changes so that we see how it was at any point in the past, not just now. First we're going to add a column to our FridgeItem table. We need to know how long it's been like it is now. Let's say I bought all these items and put them in the fridge yesterday. This our entity table with the beginnings of some history:

    FridgeItem Table:
    | Id  | Type  | State | Since when ? |
    | 1   | Milk  | Fresh | yesterday    |
    | 2   | Juice | Fresh | yesterday    |
    | 3   | Beer  | Fresh | yesterday    |

Now, I'm going to drink the beer and, while I'm reaching in for it, I notice the milk is starting to smell. What this means for the Entity table is that I'm going to `DELETE` the Beer and `UPDATE` the Milk's state to `suspicious`. But if we're not going to forget about the beers past existence and the milk's previous freshness, we're going to need a extra place to store our history. Here it is with that exampled played out.

    FridgeItem Table:
    | Id  | Type  | State      | Since when ? |
    | 1   | Milk  | Suspicious | today        |
    | 2   | Juice | Fresh      | yesterday    |

    FridgeItem_History Table:
    | Id  | Type  | State      | Since when ? | Until when ? |
    | 1   | Milk  | Yesterday  | yesterday    | today        |
    | 3   | Beer  | Fresh      | yesterday    | today        |

This sort of table structure is enough information know when every single change happened and to figure out what the contents of my Fridge was at any point of time. Right now, the current state is of course most obvious (it's just the FridgeItem table), but the query to find out what it was like before I reached in for that beer ends up being pretty easy to write. There will be examples of how to do this, and more complex queries in the coming articles.

If knowing the complete change history of your Entities is something that interests you, and you're keen to know more specifically how to setup a Temporal Table and start querying its history, the check out the next <a href= "{{ site.url }}/articles/2020-04/temporal-tables-part-2">part of this series</a>. I'll be looking at the setting up the PostgresSQL Extension implementation [arkhipov/temporal_tables](https://github.com/arkhipov/temporal_tables).
