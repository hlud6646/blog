+++
title = 'Store Simululation'
date = 2025-07-16T12:04:03+10:00
draft = false
tags = ["Postgres", "Docker", "Microservices", "AWS"]
+++

Source: [https://github.com/hlud6646/storeSimul3](https://github.com/hlud6646/storeSimul3)  
Dashboard: [https://hlud.xyz/store](https://hlud.xyz/store/)

In an actual application, there's a good chance that the interaction with the database
will take place in a format which is different to simply executing statements inside
a console. Like, it's all well and good to be able to write a `SELECT` with an 
`INNER JOIN`, but if you're connecting to the database in your Java app,
what are you going to do? This project started off as a way to explore this problem. 
It became an exercise in using docker for managing microservices and AWS for deployment.


# The Idea
The idea I had was to build a database for an imaginary store with products, suppliers, 
customers etc. 
To mimic the operations of the store, each aspect would have 
an associated microservice that periodically creates items. For example, there 
would be a Python process that picks a random customer from the database every few minutes, 
and creates an order with some randomly chosen products.
It worked, and you can see it [here](https://hlud.xyz/store).

## Initialisation and Triggers
Before anything can interact with the db, it needs to be built.
There are official postgres docker container images that let you spin up a database very 
easily. Incidentally I recommend this approach to anyone looking to learn about using a database, 
since pulling a container from dockerhub is a lot easier than configuring a db service 
to run on your own. This project has two simple `.sql` scripts: one that creates the tables
and another that creates a function and then binds it to a trigger.
The tables are nothing very interesting, just the usual auto generated Ids, some timestampz and
some obvious foreign key constraints.
The triggers are pretty pointless, but I wanted a good reference for this common and useful 
pattern. One trigger creates a new order (a 'welcome gift') each time a new customer is inserted.
The other one records any inventory modification to catch naughty employees.


## Microservices
The components of the store that are active (at time of writing) are:

| Microservice | Language | Database framework |
| --------------- | --------------- | --------------- |
| Products | Scala | JDBC |
| Suppliers | Haskell | Postgres-simple |
| Customers  | Python | SQLAlchemy |
| Orders | Elixir | Ecto |
| Api | Python | Psycopg | 

Having so many different languages in a tiny project is a weird thing to do, but I like
to compare different tools and it's been a good chance to learn.
The [dashboard](https://hlud.xyz/store) has more detail about each service, and how 
the different patterns of connecting to a database (like plain queries vs. more fancy ORM
abstractions).
Each does something fairly similar though, which is essentially to connect, and then 
periodically write made up things to the database.

## ORM vs. SQL
Two of the components interact with the database through an 'object relational mapping'
(ORM) which is an abstraction on raw SQL queries that aims to bring the experience closer
to those languages natural style.  Essentially you create a network of classes that model
the required tables in the database and the relationships between them. For a small project
the added complexity is absolutely not worth the trouble. Whether or not this is a good idea
in a large project is one which I am not qualified to tackle.



## Dockerizing

I had a lot of type-2 fun figuring out the best way to choose base images and write
dockerfiles. I started with an alpine base image, and then language specific images
that would inherit from base, so 'base-python' and 'base-haskell' etc. 
Then I had the actual services build from these. This was a mess, so I switched to one
giant dockerfile that ran everything inside one container. This was even worse, since 
a change to any part of the project that was near the top of the dockerfile would trigger
a full rebuild for everything that followed. In other words, I learnt about *layer caching* and 
went back to individual builds for each service. This time though, I was ready to 
write beautiful dockerfiles that fully leverage the power of the build cache.

For a simple Scala project, the dockerfile has two main jobs: build the project, and run the final
application. To build it, you need the full Java development kit and Scala compiler. 
These are both large pieces of software. To run it, all you need is the Java runtime 
environment which is comparatively smaller.
So a good dockerfile will work in two stages: first a 'builder' will install all the 
large development tools and compile a single jar that contains the project and all its
dependencies. Then a 'runner' stage can grab this jar and *nothing else* and run 
it in the JRE.

If you do it right, docker will compile your dependencies and cache that
layer. That way if you only change your source, the dependencies are still clean and 
don't need to be rebuilt. If you do it wrong, any tiny change and you're waiting for the 
dependencies to download and compile again. In a Haskell project this can be a LONG time.

A similar workflow exists for the orders service, which is an Elixir program. Like Java, 
Elixir runs on a virtual machine (the BEAM VM originally built for Erlang).
As above, you build and run your project in separate stages. But the Elixir build tool (`mix`)
takes it one step further by supplying the `release` command, which bundles your final compiled 
project with its dependencies *and the BEAM runtime itself*, giving you what looks like a 
native executable that you can run on a lightweight image. I found this quite cool.



## Deploying
I tried Fly.io with the single container build, but it bothered me that if nobody had 
visited in the last little while (i.e. all the time) the server response was really slow. This is the consequence of
a useful feature that keeps cost down but I thought I could do better using free stuff on AWS.
In short the whole thing (including the database) runs on a single EC2 instance. Being a pretend webmaster has been a 
useful experience in its own right and I'll probably write about it in another post.

## Wrapping Up
This has been a great project for learning a surprisingly broad range of tools.
What started as an attempt to try some practical database exercises presented an
opportunity to use advanced features of Docker, explore automated deployment options
and finally to practice being a webmaster on AWS.
