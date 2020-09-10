---
layout: post
title: "Using parameterized migrations with SimpleMigrator"
date: 2020-09-09 12:00:00
excerpt: "How to pass parameters to your Migration Classes"
categories: [storage]
comments: false
---


[Simple.Migrations](https://github.com/canton7/Simple.Migrations) is a very handy migration framework for .NET Core. I've used it many times. Writing migrations is very easy and for each one you end up with class like this.

    [Migration(1, "Do Something")]
    public class CreateSomeStuff : Migration
    {
        protected override void Up()
        {
            Execute("SOME SQL TO create.stuff");
        }

        protected override void Down()
        {
            Execute("SOME OTHER SQL to destroy.stuff");
        }
    }

But. What if you want a `Migration` that has a constructor with parameters?

Admittedly this is not a super common situation, but I have come across it. I needed to re-use my database migration code to standup an identical set of tables in another schema, so that I could use it as a staging area to rebuild my database from an event store. The task was made a bit complex by some shared state between schemas (Operator classes and extensions), so the migrations needed to know what was going on. Your reason may be different, but we both want a Migrator that looks something like this:

    ...
    public class CreateSomeStuff : Migration
    {
        public CreateSomeStuff(Settings settings)
        {
            _settings = settings;
        }
        ...
    }


This is not supported out of the box. If you try it the migrator will throw because it expects a parameterless constructor. But it's pretty easy to extend the `SimpleMigrator` to support parameters. Like this:

    public class ParameterizedSimpleMigrator<TMigrationParameter> : SimpleMigrator<DbConnection, IMigration<DbConnection>>
    {
        private readonly TMigrationParameter _parameter;

        public ParameterizedSimpleMigrator(IMigrationProvider migrationProvider, IDatabaseProvider<DbConnection> databaseProvider,
            TMigrationParameter migrationParameter, ILogger logger = null) : base(migrationProvider, databaseProvider,
            logger)
        {
            _parameter = migrationParameter;
        }

        public ParameterizedSimpleMigrator(Assembly migrationsAssembly, IDatabaseProvider<DbConnection> databaseProvider,
            TMigrationParameter migrationParameter, ILogger logger = null) : base(migrationsAssembly,
            databaseProvider, logger)
        {
            _parameter = migrationParameter;
        }

        protected override IMigration<DbConnection> CreateMigration(MigrationData migrationData)
        {
            if (migrationData == null)
                throw new ArgumentNullException(nameof(migrationData));

            IMigration<DbConnection> instance;
            try
            {
                instance = (IMigration<DbConnection>)
                    Activator.CreateInstance(migrationData.TypeInfo.AsType(), args: new object[] { _parameter });
            }
            catch (Exception e)
            {
                throw new MigrationException($"Unable to create migration {migrationData.FullName}", e);
            }

            return instance;
        }
    }

The only change for the vanilla SimpleMigrator is that I've changed the `Activator.CreateInstance` to pass one constructor parameter of generic type `TMigrationParameter` to the Migrations, which is an additional value that needs to be passed into `ParameterizedSimpleMigrator`'s constructor.

Just start using this instead of the `SimpleMigrator` class, and you can add all the parameters to your migrations you like.
