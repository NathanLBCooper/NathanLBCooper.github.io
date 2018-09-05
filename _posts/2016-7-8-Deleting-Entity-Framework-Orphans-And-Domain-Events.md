---
layout: post
title: "Deleting Entity Framework Orphans and Domain Events in Entity Framework 6"
date: 2016-07-08 12:00:00
categories: [entity-framework-6, how-to]
comments: false
---

One of the 'missing features' of Entity Framework 6 is properly dealing with the removal of entities from collections. A naive `listOfEntities.Remove()` won't actually delete the entity. Instead it will remove the relation between the entity and the collection. This might burn you in a very obvious way if you have constrained the foreign key of the child element to not be null.

This also means that cascade deletes you may have set up won't work. Although there are more reasons they might not work anyway. Jimmy Bogard provides a workaround for this on his site, lostechies.com: [Missing EF Feature Workarounds: Cascade delete orphans](https://lostechies.com/jimmybogard/2014/05/08/missing-ef-feature-workarounds-cascade-delete-orphans/).

This post fixing Entity Framework's deletion problem using Domain Events, which you can also read about on [lostechies.com](https://lostechies.com/jimmybogard/2010/04/08/strengthening-your-domain-domain-events/).

I think this a good idea example of domain events, which are a useful tool for building applications. This isn't actually my code, a ex-colleague of mine wrote pretty much all of this, but I thought it would be a useful thing to share here.

##### Domain Events

First things first, let's create interfaces for the raising and handling of events:

```csharp

public interface IDomainEventHandler<in T> where T : IDomainEvent
{
    void Handle(T domainEvent);
}

public interface IDomainEvent
{
}

```

And a class to orchestrate our domain event handling. This class keeps track of the IDomainEventHandlers and routes the IDomainEvents to them based on type.

```csharp

public static class DomainEvents
{
    private static readonly Logger Log = LogManager.GetCurrentClassLogger();

    [ThreadStatic] //so that each thread has its own callbacks
    private static Lazy<List<Delegate>> domainEventHandlers;

    static DomainEvents()
    {
        domainEventHandlers = new Lazy<List<Delegate>>();
    }

    // Registers a callback for the given domain event
    public static void Register<T>(Action<T> callback) where T : IDomainEvent
    {
        domainEventHandlers.Value.Add(callback);
    }

    // Clears callbacks passed to Register on the current thread
    public static void ClearCallbacks()
    {
        domainEventHandlers = new Lazy<List<Delegate>>();
    }

    // Raises the given domain event
    public static void Raise<T>(T domainEvent) where T : IDomainEvent
    {
        // Find any registered handlers in the IoC container via ServiceLocator
        if (ServiceLocator.IsLocationProviderSet)
        {
            Log.Trace("Domain event '{0}' raised", typeof (T).Name);

            foreach (var handler in ServiceLocator.Current.GetAllInstances<IDomainEventHandler<T>>())
            {
                Log.Trace("Handler '{0}' starting", handler.GetType().Name);

                handler.Handle(domainEvent);

                Log.Trace("Handler '{0}' finished", handler.GetType().Name);
            }

            Log.Trace("Domain event '{0}' handled", typeof (T).Name);
        }

        if (domainEventHandlers == null)
        {
            return;
        }

        Log.Trace("Domain event '{0}' raised", typeof (T).Name);

        foreach (var action in domainEventHandlers.Value.OfType<Action<T>>())
        {
            action(domainEvent);
        }

        Log.Trace("Domain event '{0}' handled", typeof (T).Name);
    }
}


```

##### Entity Framework Specifics

Now to create the specific code for the entity framework stuff. `Entity` being our base class for the thing we'd like to delete, although this could be `object`.

```csharp

public class EntityDeleted : IDomainEvent
{
    public Type EntityType { get; private set; }
    public Entity DeletedEntity { get; private set; }

    public EntityDeleted(Entity deletedEntity)
    {
        EntityType = deletedEntity.GetType();
        DeletedEntity = deletedEntity;
    }
}

```

And here is the handler. Our datacontext is `IBusinessDataContext`, which inherits from `DbDataContext`.


The `Handle(EntityDeleted domainEvent)` is where we actually do the deletion.
The `IObjectContextAdapter` is an Entity Framework-ism we use to get the underlying `ObjectContext` from the `DataContext`, exposing the `DeleteObject` method we need.


```csharp

public class EntityDeleteHandler : IDomainEventHandler<EntityDeleted>
{
    private readonly IBusinessDataContext _context;
    private readonly IObjectContextAdapter _contextAdapter;

    public EntityDeleteHandler(IBusinessDataContext context)
    {
        _context = context;
        _contextAdapter = (IObjectContextAdapter)_context;
    }


    public void Handle(EntityDeleted domainEvent)
    {
        ObjectStateEntry state;
        if (_contextAdapter.ObjectContext.ObjectStateManager.TryGetObjectStateEntry(domainEvent.DeletedEntity, out state))
        {
            if (state.State != EntityState.Detached)
                _contextAdapter.ObjectContext.DeleteObject(domainEvent.DeletedEntity);
        }
    }
}

```

Here's an example usage:

```csharp

if (Products.Contains(product))
{
	// Remove from the collection
	//(which deletes the relationship rather than the entity)
    ProductsCollection.Remove(product);

	// Call the domain event,
	// allowing EntityDeleteHandler (if registered) to actually delete the object
    DomainEvents.Raise(new EntityDeleted(product));
}

```

NB:

These events are dispatched before commit, so if handler does something outside of the entity framework transition scope (like send an email), it will still happen if there’s a rollback. This isn't a problem for my example usage, but either a more sophisticated event handling mechanism, or some kind of distributed transaction scope would be needed.
