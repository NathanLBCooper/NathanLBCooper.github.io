---
layout: post
title: "Building your own event logging"
date: 2016-07-11 12:00:00
excerpt: "Let's make our logging structured"
categories: [programming]
comments: false
image:
  feature: code.png
---

> NOTE: I think these days it's better to just take something like ELK and Prometheus and just do it how they say. But here is how I used to do event logging:

Let's say we're writing a system that sends messages, then it logs: `Log($"Message sent to {0}", value.SentTo, LogLevel.DEBUG)`. So when Debug level events are being logged, we get a nice string explaining what happened. But what's wrong with this?

**It's hard to consume/automate.** The strong description of what this event actually means and the  structure of the data it holds has been lost at the point of logging it. Getting this back from the string is going to be difficult and fragile. Automating the reading of the logs is now much harder than it needs to be. Doing other interesting things, like powering UI from the logging of events, is also difficult here.

**Your code is probably inconsistent/repetitive and could be more expressive.** I claimed that our imaginary system logged `"Message sent to {0}"` each time a message was sent. I lied, it probably logs a half dozen variations of the exact same thing with a dozen different places. Writing something like `Log(new MessageSentEvent(value))` gets rid of that duplication. I think it's clearer code as well.

## SLAB ##

These are killer arguments for Semantic Logging, and you can read more about them on the page for Microsoft's [Semantic Logging Application Block (or SLAB)](https://msdn.microsoft.com/en-us/library/dn775014(v=pandp.20).aspx). Semantic Logging is great and I'm glad Microsoft are embracing it, **it's just too bad that SLAB sucks.**

I want to spend more time on saying how to do it right than how Microsoft did it wrong. But let's look at a simple example from a [SLAB tutorial](https://msdn.microsoft.com/en-us/library/dn440729(v=pandp.60).aspx)

```csharp

MyCompanyEventSource.Log.ScalingRequestSubmitted(
    request.RoleName, 
    request.InstanceCount,
    context.RuleName,
    context.CurrentInstanceCount);

```

**Wrong.** I don't like that I have to edit `MyCompanyEventSource` every time I add a new event type, and the static logger makes it impossible to test what a class logs.

Also, if we look inside `MyCompanyEventSource` we get method like this:

```csharp

    [Event(3, Message = "loading page {1} activityID={0}",
    Opcode = EventOpcode.Start,
    Task = Tasks.Page, Keywords = Keywords.Page,
    Level = EventLevel.Informational)]
    internal void PageStart(int ID, string url)
    {
      if (this.IsEnabled()) this.WriteEvent(3, ID, url);
    }

```

This code is obviously ugly. Attributes cannot be the right choice here. I don't believe that logging is such a complex concern that all of this is necessary. I'm not interested in defining all my events in one big long class. There must be a better way.

## So how do we do this right? ##

First things first. I'll be using C# and [Nlog](http://nlog-project.org/) to demonstrate the "right way"™ , but the principal is language agnostic. Sure, [I've written this in C# before](https://github.com/NathanLBCooper/ProcessGremlin/tree/master/Logging), but I've also written [basically the same thing in C++ as well](https://github.com/NathanLBCooper/ableton-freetime-looper/tree/master/LiveFreetimeLooper.Core/Logging)
 
The point of event logging is to give event structure and meaning. It makes sense to create a interface for this.

```csharp

using System;
using NLog;

namespace Logging
{
    public interface IEvent
    {
        string Name { get; }
        DateTime Time { get; }
        string Detail { get; }
        LogLevel Level { get; }
        string EventSource { get; }
    }
}

```

We'll create concrete types to represent actual events in a moment, but let's create something that handles these events first.

```csharp

namespace Logging
{
    public interface IEventLogger
    {
        void Log(IEvent evt);
    }
}

```

And here's an actual implementation of an `IEventLogger`, which uses NLog.

```csharp

using System.Collections.Concurrent;
using NLog;

namespace Logging
{
    public class EventLogger : IEventLogger
    {
        private static readonly ConcurrentDictionary<string, Logger> Loggers = new ConcurrentDictionary<string, Logger>();

        public void Log(IEvent evt)
        {
            var logger = GetLogger(evt);
            logger.Log(evt.Level, string.Format("{0} : {1} : {2}", evt.Time, evt.Name, evt.Detail));
        }

        private static Logger GetLogger(IEvent evt)
        {
            return Loggers.GetOrAdd(evt.EventSource, LogManager.GetLogger);
        }
    }
}

```

Now all we have to do to extend this is create `IEvents`. For example, this is a rough idea of how we'd reproduce SLAB's PageStart Event. The event declaration:

```csharp

namespace Logging.Events
{
    public sealed class PageStartEvent : IEvent
    {
        public PageStartEvent(int id, Url url, Type source)
        {
            Detail = $"loading page {url.ToString()} activityID={id}";
            Level = LogLevel.Info;
            Name = "Page Starting";
			Source = source;
        }
    }
}

```

And its use:

```csharp

_eventLogger.Log(new PageStartEvent(id, url, this.GetType()));

```


So that's how, with two interfaces and bit of code specific to your particular logging framework, we can get all the benefits of semantic logging.

Note that some detail, specific to particular events (like the `id` and `url` in our example) is still lost into strings, but now we have each event neatly represented by a type, it wouldn't be hard to create something like a `JsonDetails` property, a `Dictionary<string,string>` property. Alternatively you could subclass / create alternatives to `IEvent` and `IEventLogger` for specific things you want to monitor. I would check out [Martin Fowler's article on Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) if you're logging starts to become this sort of detailed store of changes.