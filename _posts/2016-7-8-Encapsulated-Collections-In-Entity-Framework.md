---
layout: post
title: "How to use properly encapsulated collections In Entity Framework"
date: 2016-07-08 12:00:00
categories: [entity-framework-6, how-to]
comments: false
---

This is the pattern I use when writing classes with Collections in Entity Framework 6 (EF). Specifically, when these EF entities need to be proper domain objects with proper encapsulation, rather than just lightweight data objects. 

The reason proper encasulation is tricky, is that the collections you expose have to be powerful. They need to expose the ability to add and remove items, because EF needs to be able to do that
That's great for EF, but it might just break the encapsulation of your class. Here is the pattern I use to expose the correct collection interface to consumers of my class, while accommodating Entity Framework.

I used a backing collection with just the level of access EF needs: 

```csharp

protected internal virtual ICollection<MyEntity> MyEntitysCollection { get; set; }

```

I expose that collection as something less powerful. As an `IEnumerable` or a `IReadonlyCollection`, for example. Or maybe I don't expose it at all. I follow my class design instincts without caring about EF.

```csharp

public IEnumerable<MyEntity> MyEntities
{
	get
	{
		return MyEntityCollection;
	}
}

```

Here's the magic bit. When creating the entity type configuration I'll need access the actual collection. But instead of ruining our encapsulation by making it public, I can do this instead:

```csharp

public static Expression<Func<MyClassThisIsIn, ICollection<MyEntity>>> MyEntityMapping =
	f => f.MyEntityCollection;

```

Which enables me configure EF like this:

```csharp
	
// Example
HasMany(MyClassThisIsIn.MyEntityMapping)
            .WithRequired()
            .HasForeignKey(e => e.MyClassThisIsIn_Id);

```

Done. Now I have a class with a collection that has the correct encapsulation, but also works as an Entity Framework entity.
