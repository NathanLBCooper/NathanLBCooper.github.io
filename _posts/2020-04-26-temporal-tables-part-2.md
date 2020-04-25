---
layout: post
title: "Temporal Tables Part 2: Setup Database and Create Tables"
date: 2020-04-25 12:01:00
excerpt: "How to setup temporal tables in PostgresSQL"
categories: [storage]
comments: false
---

We're going to set up [arkhipov's Temporal Tables PostgresSQL Extension](https://github.com/arkhipov/temporal_tables) this article, so that we can start working with it in the next part of the series. If you're following this series and creating your own temporal tables along side me, I think that's fantastic. But know that these articles are no substitute for reading the project's README, which is full of useful setup and troubleshooting tips. I'm going to be running through things as quick as I can here, and leave things like edge-cases or customizations to the README.

I'm going to use Docker and Docker compose in this article. That doesn't mean you have to, but I feel it's a good way for me to communicate exactly what's going on and give you something you can easily and exactly reproduce.

Okay, that's all said, so let's start.

### Let's create the database and install the extension

I'm going to create a new database from scratch with the necessary files for this extension in place, but it is also possible add them to existing database ([documentation here](https://www.postgresql.org/docs/current/extend-extensions.html)). Here's my Dockerfile, which clones and builds the extension into *postgres:10-alpine*:

    FROM postgres:10-alpine

    RUN apk add --no-cache --virtual build-deps musl-dev make gcc git && \
        git clone https://github.com/arkhipov/temporal_tables.git --depth 1 && \
        cd temporal_tables && \
        make && \
        make install && \
        cd ../ && rm -rf temporal_tables && \
        apk del build-deps musl-dev make gcc git

Then let's setup a docker compose file. I'm going add "pgadmin" as well, it's a tool that will let us connect to and look at our database. I've put my postgres Dockerfile in a folder `infrastructure/postgres/` relative to this file and `.data` is file I store docker stuff in. I've configured what needed to be configured for [dockerised PostgresSQL](https://hub.docker.com/_/postgres). There's some stuff that you may want to configure differently, and that's just fine.

    version: '2.4'

    services:
        postgres:
        image: ${DOCKER_REGISTRY-}mytemporaldb
        build:
        context: .
        dockerfile: infrastructure/postgres/Dockerfile
        args:
            - VERSION=0.2.0
        expose:
            - 5432
        ports:
            - "5432:5432"
        mem_limit: 512M
        restart: always
        environment:
            POSTGRES_DB: temporaldb
            POSTGRES_USER: temporalsqluser
            POSTGRES_PASSWORD: temporalsqlpassword
            PGDATA: /var/lib/postgresql/data
        volumes:
            - pgdata:/var/lib/postgresql/data:rw
        healthcheck:
            test: ["CMD-SHELL", "pg_isready -U postgres"]
            interval: 30s
            timeout: 10s
            retries: 3

    pgadmin:
        image: dpage/pgadmin4
        ports:
        - "5050:80"
        depends_on:
        - postgres
        mem_limit: 512M
        restart: always
        environment:
            PGADMIN_DEFAULT_EMAIL: login@example.com
            PGADMIN_DEFAULT_PASSWORD: password
        volumes:
      - .data/pgadmin:/root/.pgadmin

    volumes:
        pgdata:

Finally to install the extension you just connect to the database as a super user and run this:

    `CREATE EXTENSION temporal_tables;`

Then we're done. The database exists and all we have to do is start using the extension.

### It's time to create some tables

First create we create a table for our entity. Let's use Fridge Items again.

    CREATE TABLE public.FridgeItem
    (
        Id     integer  PRIMARY KEY,
        Type   text     NOT NULL,
        State  text     NOT NULL
    );

In order to make this temporal table we add a system period column:

    ALTER TABLE FridgeItem ADD COLUMN sys_period tstzrange NOT NULL;

Then we add a history table that will contain the archived rows of our table:

    CREATE TABLE public.FridgeItem_History (LIKE public.FridgeItem);

And add a trigger to connect them

    CREATE TRIGGER fridgeitem_versioning_trigger
    BEFORE INSERT OR UPDATE OR DELETE ON public.FridgeItem
    FOR EACH ROW
    EXECUTE PROCEDURE versioning(
        'sys_period', 'FridgeItem_History', true
    );

And, that's it. The setup is done.

### Now let's make some changes and see that history created

We'll run the almost the same example from the part 1, except this time it's real. Just to remind ourselves, I first added Milk, Juice and Beer into my Fridge.

```SQL
INSERT INTO public.FridgeItem (Id, Type, State)
VALUES (1, 'Milk', 'Fresh');

INSERT INTO public.Entity (EntityId, Value)
VALUES (2, 'Juice', 'Fresh');

INSERT INTO public.Entity (EntityId, Value)
VALUES (3, 'Beer', 'Fresh');
```
Now the FridgeItem table contains these values and the FridgeItem_History table is empty. Todays date is `2020-04-26` and you'll notice that appears in the sys_period column.

    FridgeItem:
    Id | Type    | State   | sys_period
    -- | ------- | ------- | --------------
    1  | 'Milk'  | 'Fresh' | [2020-04-26, )
    2  | 'Juice' | 'Fresh' | [2020-04-26, )
    3  | 'Beer'  | 'Fresh' | [2020-04-26, )

    FridgeItem_History:
    Id | Type    | State   | sys_period
    -- | ------- | ------- | --------------
       |         |         |

For the purpose of this example, time travel forward to tomorrow (`2020-04-27`). Now it's time to drink the beer and notice that the milk is going bad.

```SQL
UPDATE public.FridgeItem SET State = 'Suspicious' WHERE Id = 1;

DELETE FROM public.FridgeItem where Id = 3;
```

Now the the table still has the correct data for the present time, but we also have history. The beer is gone and the milk has been changed, but their old values are now recorded in our FridgeItem_History table and the sys_period shows when the historic values used to be the current values.

    FridgeItem:
    Id | Type    | State   | sys_period
    -- | ------- | ------- | --------------
    1  | 'Milk'  | 'Suspicious' | [2020-04-27, )
    2  | 'Juice' | 'Fresh' | [2020-04-26, )

    Entity_History:
    Id | Type    | State   | sys_period
    -- | ------- | ------- | --------------
    1  | 'Milk'  | 'Fresh' | [2020-04-26, 2020-04-27)
    3  | 'Beer' | 'Fresh'  | [2020-04-26, 2020-04-27)

We'll talk about how to start querying that <a href= "{{ site.url }}/articles/2020-04/temporal-tables-part-3">next time</a>.