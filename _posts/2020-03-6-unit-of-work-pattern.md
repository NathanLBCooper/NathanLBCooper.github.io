---
layout: post
title: "How to implement a Unit of Work pattern"
date: 2020-03-06 12:01:00
excerpt: "A Unit of Work pattern for repositories using ambient context"
categories: [programming, storage]
comments: false
---

When using a repository pattern, it's a common to run into the following situation: Running one of your 'business transactions' (a login request, a customer order etc) requires multiple entities to be updated. Multiple methods on multiple different repositories must be called. A partial update, when some calls fail and some succeed is not okay. They all need to succeed or fail together.

A **Unit of Work** is a bucket that we put all these separate operations into so they can be run together. In SQL the underlying mechanism for achieving this is to use a transaction, but the concept is more general and not tied specifically to SQL.

**Here's how to implement it**. I'll be using a style of Unit of Work that allows for separately injectable repositories and avoids god objects (explained in detail at the bottom). I'll be using C# and Dapper on top of a SQLite in-memory database:

Let's start with the usage. Calls to the repositories are done within the lifetime of a `UnitOfWork`, provided by a `IUnitOfWorkContext`.

    public class MyBusinessCode {
        private readonly IUnitOfWorkContext _unitOfWorkContext;
        private readonly ClientRepository _clientRepository;
        private readonly OrderRepository _orderRepository;

        public MyBusinessCode(IUnitOfWorkContext unitOfWorkContext,
            ClientRepository clientRepository,
            OrderRepository orderRepository)
        {
            _unitOfWorkContext = unitOfWorkContext;
            _clientRepository = clientRepository;
            _orderRepository = orderRepository;
        }

        public async Task DoBusiness(int number)
        {
            using (UnitOfWork uow = context.Create())
            {
                await _clientRepository.CreateAsync(number);
                await _orderRepository.CreateAsync(number);
                await uow.CommitAsync();
            }
        }
    }

What is this `UnitOfWork`? In this case it just wraps a transaction. It can be committed or rolled back.

    public class UnitOfWork : IDisposable
    {

        private readonly SQLiteTransaction _transaction;
        public SQLiteConnection Connection { get; }

        public bool IsDisposed { get; private set; } = false;

        public UnitOfWork(SQLiteConnection connection)
        {
            Connection = connection;
            _transaction = Connection.BeginTransaction();
        }

        public async Task RollBackAsync()
        {
            await _transaction.RollbackAsync();
        }

        public async Task CommitAsync()
        {
            await _transaction.CommitAsync();
        }

        public void Dispose()
        {
            _transaction?.Dispose();

            IsDisposed = true;
        }
    }

And here is how a repository is implemented. The important part here is that instead of creating their own connections, the repository gets it from an `IConnectionContext`.

    public class ClientRepository
    {
        private readonly IConnectionContext _context;

        public ClientRepository(IConnectionContext context)
        {
            _context = context;
        }

        public async Task<int> CreateAsync(int value)
        {
            return await _context.GetConnection().QuerySingleAsync<int>(
                @"
    insert into Client (Value) values (@value);
    select last_insert_rowid();
    ", new { value });
        }

        public async Task<Client> GetOrDefaultAsync(int id)
        {
            return await _context.GetConnection().QuerySingleOrDefaultAsync<Client>(
                @"
    select * from Client where Id = @id
    ", new { id });
        }
    }

But what is this `IConnectionContext` that gives us these connections, and what was that `IUnitOfWorkContext` that let us get a `UnitOfWork`? They are two interfaces for the same thing, the `UnitOfWorkContext`. This is the object that ties everything together and makes sure the SQL in the repositories is run in the unit of work requested by the business code.

    public interface IConnectionContext
    {
        SQLiteConnection GetConnection();
    }

    public interface IUnitOfWorkContext
    {
        UnitOfWork Create();
    }

    public class UnitOfWorkContext : IUnitOfWorkContext, IConnectionContext
    {
        private readonly SQLiteConnection _connection;
        private UnitOfWork _unitOfWork;

        private bool IsUnitOfWorkOpen => !(_unitOfWork == null || _unitOfWork.IsDisposed);

        public UnitOfWorkContext(SQLiteConnection connection)
        {
            _connection = connection;
        }

        public SQLiteConnection GetConnection()
        {
            if (!IsUnitOfWorkOpen)
            {
                throw new InvalidOperationException(
                    "There is not current unit of work from which to get a connection. Call BeginTransaction first");
            }

            return _unitOfWork.Connection;
        }

        public UnitOfWork Create()
        {
            if (IsUnitOfWorkOpen)
            {
                throw new InvalidOperationException(
                    "Cannot begin a transaction before the unit of work from the last one is disposed");
            }

            _unitOfWork = new UnitOfWork(_connection);
            return _unitOfWork;
        }
    }

The full version is available on github here: [https://github.com/NathanLBCooper/unit-of-work-example](https://github.com/NathanLBCooper/unit-of-work-example)

----------

**What if I don't want to use an ambient context? Can I link the unit of work and the repositories in some other way?**

Yes you could. If you're familiar with the Entity Framework `DbContext` unit of work, you may have noticed a difference as soon as you read the usage. In Entity Framework the `DBContext` both contains the generic repositories and is the unit of work. There is no such object in my implementation. But it's actually pretty common to implement a Unit of Work pattern that way.

Here's what the usage would look like if we took some responsibilities away from the `IUnitOfWorkContext`, renamed it `DBContext` and then made the `UnitOfWork` responsible for the repositories:

    public class MyBusinessCode {
        private readonly DBContext _dbContext;

        public MyBusinessCode(DBContext dbContext) { ... }

        public async Task DoBusiness(int number)
        {
            using (UnitOfWork uow = context.Create())
            {
                await uow.ClientRepository.CreateAsync(number);
                await uow.OrderRepository.CreateAsync(number);
                await uow.CommitAsync();
            }
        }
    }

This is simpler than my implementation in a number of ways. The ambient joining magic of the `UnitOfWorkContext` is gone and it's now a compile time restriction that the repositories be used just within a Unit of Work. It's also harder to screw up using this when multi-threading. That's because a drawback of the `UnitOfWorkContext` is that it holds state about what Unit of Work is happening, so you have to make sure that object lives in the correct scope. `IDBContext` doesn't hold this state and can be shared with less caution.

But there's one massive drawback to this alternative approach. The Unit of Work has become a God object of repositories. `MyBusinessCode` cannot express clearly what it's here to change via its dependencies. Because it's always one `.` away from changing anything anywhere in the database. God objects like this seem innocuous at first but can fairly quickly lead to code chaos.

Which ones of these two patterns depends on what you value and how much you're willing to pay to avoid god objects.
