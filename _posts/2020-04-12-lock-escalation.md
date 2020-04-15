---
layout: post
title: "Why is a SQL DELETE statement locking the entire table?"
date: 2020-04-12 12:01:00
excerpt: "And why do I keep seeing 'DELETE TOP 4000 FROM'? A quick summary of lock escalation in SQL Server"
categories: [storage]
comments: false
---

A transaction will lock the resources (rows, pages, tables etc) on which it is dependent. It does this to prevent other transactions from simultaneously modifying them and causing problems for the original transaction. Ideally a transaction will lock only what's needed, but that's not always the case.

SQL can lock on multiple things. Here's something called the "Lock Hierarchy", from smallest to biggest:

> Row  <  Page  <  Table  <  Database

Let say I'm modifying 1000 rows. SQL will take the same number of row locks: 1000. But if I were to modify 10 000 instead, this is no longer going to be the case. Since keeping many locks costs memory, at some point SQL will decide to keep fewer chunkier locks instead. When I modify those 10 000 rows a single table lock will be taken instead. Depending on how I use that table, that might be a problem that cripples my system until that transaction is complete.

The point at which SQL will swap many small locks out for fewer bigger ones is called the "**Lock Escalation Threshold**". It is at **5000 locks** ([more detailed definition here](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms184286(v=sql.105)?redirectedfrom=MSDN)).

So if you're experiencing table locks on your large transaction, makes sure you're staying under this threshold.

Here is how you might batch a large transaction (in this case a delete) into smaller ones that won't lock the table:

    DECLARE @deletedInBlock INT;
    SET @deletedInBlock = 1;

    WHILE @deletedInBlock > 0
    BEGIN
        DELETE TOP 4000 FROM dbo.MyTable
        SET @numberDeletedInBlock = @@ROWCOUNT;
    END
