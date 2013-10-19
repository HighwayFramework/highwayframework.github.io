---
layout: post
title: "Understanding the Patterns"
date: 2013-10-19 10:40
comments: true
no_homepage: true
order: 1
categories: data feature
---
#Patterns
Highway.Data is based on a blended use of **3 major patterns**, and to understand the intent of the Queries / Commands / Scalars that make highway smooth, we must first talk about the patterns.

There are a lot of places where you can find the academic explanation of these patterns, but here we are going to focus on the outcomes we are using it for. If you would like to read more about these, check out the [Further Reading](#furtherReading) below.

##Repository
Repository is a way to abstract the "guts" of the data access and persistence knowledge away from my logic. This is a lot of the time misused. Typed Repositories, Repositories with more than 5 methods, repositories that manage their own statefulness are all examples of a good pattern meeting a bad implementation. We use Repository in combination with Unit of Work and a modern interpretation of Query Object pattern.

###Signs of a well built Repository

* The user of the repository ask for data, but no information about the persistence is leaked to the caller.
* It is not specific to any domain, and can be portable
* It has state, but doesn't manage it
* It has no control of the Update/Add/Delete side of the operations, but gives access to that API. Repositories are meant as an abstraction for reading data. 

##Unit Of Work
Unit of Work allows us to track in memory changes of an object that are made in series and then commit those changes to some persistence layer. This is an atomic unit of work that is transaction during the commit but in our code is just changes to objects.

###Signs of a well built unit of work

* Commit-able
* Abandon/Rollback-able
* Not specific to any one API
* Give the basic Add/Remove/Update functionality
* Are not query-able

##Query Object
Query Object is a way of encapsulating the details of a single query into an object that can be reused without being stateful. This helps us avoid duplication in the code base and we can codify and test our queries.

###Signs of a well built query object

* It doesn't manager state outside a single execution
* It is reuse-able
* It is able to be functionally tested
* It is able to be performance tested
* It can output it's persistence query ( SQL statement for example )
* If it returns multiple rows, it can have pagination applied without modification

<a name="furtherReading"></a>
##Further Reading 
We hold our opinions because of years of software development on all sizes of projects, but understand that many opinions exist that are valid. In an effort to present the information as impartially as possible below does include links to content that disagrees with our opinions. Please read on and form your own opinions.



###Repository
[Martin Fowler Repository Pattern](http://www.martinfowler.com/eaaCatalog/repository.html)

[Fredik Normén Repository Pattern Purpose Discussion](http://weblogs.asp.net/fredriknormen/archive/2008/04/24/what-purpose-does-the-repository-pattern-have.aspx)

[MSDN Repository Article](http://msdn.microsoft.com/en-us/library/ff649690.aspx)

[Ayende@Rahien Repository Pit Of Doom](http://ayende.com/blog/4784/architecting-in-the-pit-of-doom-the-evils-of-the-repository-abstraction-layer)

[Devlin Liles Discussion of the Pit Of Doom](http://www.devlinliles.com/post/I-disagree-with-the-pit-of-Doom)

###Unit of Work

[Martin Fowler Unit of Work](http://www.martinfowler.com/eaaCatalog/unitOfWork.html)

[Jeremy Miller Unit of Work and Persistence Ignorance](http://msdn.microsoft.com/en-us/magazine/dd882510.aspx)

[Ruby Lacovara Entity Framework Patterns Unit of Work](http://rlacovara.blogspot.com/2009/04/entity-framework-patterns-unit-of-work.html)


###Query Object

[Martin Fowler Query Object](http://martinfowler.com/eaaCatalog/queryObject.html)

[Karl Nilsson Query Object Pattern](http://coderkarl.wordpress.com/2012/05/02/the-query-object-pattern-2/)