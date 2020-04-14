---
layout: post
title: "Why is a SQL DELETE statement locking the entire table?"
date: 2020-04-12 12:01:00
excerpt: "And why do I keep seeing 'DELETE TOP 4000 FROM'? A quick summary of lock escalation in SQL Server"
categories: [storage]
comments: false
image:
  feature: storage.png
---

A transaction will lock the resources (rows, pages, tables etc) on which it is dependent. It does this to prevent other transactions from simultaneously modifying them and causing problems for the original transaction. Ideally a transaction will lock exactly what it needed for the time it needed, but that's not always the case.

Here's something called the "Lock Hierarchy", from smallest to biggest:

> Row  <  Page  <  Table  <  Database

When deleting or modifying 1000 rows. SQL will take 1000 row locks. But if the number of rows increases to 10 000, this is is no longer true. Keeping locks costs memory and SQL will have decided to keep fewer chunkier locks instead. Deleting 10 000 rows will lead to 1 table lock instead. Depending on how you use the table, locking rows might be fine, but locking a table might cripple your system until that transaction is complete.

The point at which SQL will swap many small locks out for a bigger one is called the "**Lock Escalation Threshold**" and is at **5000 locks**. ([usually, exact definition here](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms184286(v=sql.105)?redirectedfrom=MSDN)).

So if you're experiencing table locks on your large transaction, makes sure you're staying under this threshold.

Here's how you might batch a large transaction (in this case a delete) into smaller ones that won't lock the table:

    DECLARE @deletedInBlock INT;
    SET @deletedInBlock = 1;

    WHILE @deletedInBlock > 0
    BEGIN
        DELETE TOP 4000 FROM dbo.MyTable
        SET @numberDeletedInBlock = @@ROWCOUNT;
    END