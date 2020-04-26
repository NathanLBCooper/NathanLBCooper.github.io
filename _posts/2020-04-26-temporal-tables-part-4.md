---
layout: post
title: "Temporal Tables Part 4: Bend time"
date: 2020-04-26 13:01:00
excerpt: "The temporal tables we've been using have been 'system-period temporal tables'. Which means that all the times are based on the time of the database system when the rows are updated. But it's also possible to use custom times."
categories: [storage]
comments: false
---

By default, arkhipov/temporal_tables will use `CURRENT_TIMESTAMP` to record the times things change.

But let's say we have an application sitting at then end of queue, asynchronously processing events that happened earlier. The system time of our database when are rows are changed isn't useful here. We need to bend that system time to back when the business changes we're recording actually happened. Luckily we can do that easily.

The trick is to run:

    SELECT set_system_time('1985-08-08 06:42:00+08');

Then all of the changes we make will be recorded as happening at that time. Until we set it back to default by calling:

    SELECT set_system_time(NULL);

If `set_system_time` is set during a transaction that is aborted, it will be aborted alongside the transaction. If it's committed, it will last until the end of the session. It's up to you to make sure that you don't burn yourself by forgetting to unset it.

I use C# and Dapper. I'm going to show you a simplified version of how I write my time code, using those tools:

    public class TemporalTableTimeScope : IDisposable
    {
        private readonly NpgsqlConnection _connection;

        public TemporalTableTimeScope(DateTime customTime, NpgsqlConnection connection)
        {
            _connection = connection;
            _connection.Execute(@"SELECT set_system_time(@customTime);",
                new {customTime = customTime.ToUniversalTime()});
        }

        public void Dispose()
        {
            _connection.Execute(@"SELECT set_system_time(NULL);");
        }
    }

    // Usage:
    using (new TemporalTableTimeScope(customTime, connection))
    {
        _repo.UpdateStuff(); // These changes are persisted as if they happened at customTime
    }

And it's easy as that. With a small amount of code we can now bend time and use temporal tables in our asynchronous systems.

One thing worth noting though. Order is still important. You can't use this mechanism to add historic data if newer changes have already been made. So more strictly speaking we can use temporal tables for our *ordered* asynchronous processing. How to deal with out of order or historic changes is something I'll deal with in a later article.

