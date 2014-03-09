---
layout: post
title: "Domain Context"
date: 2014-01-25 16:42
comments: true
no_homepage: true
categories: data feature
---
#Domain Context and Domain Driven Design (DDD) Patterns
##Why did you support DDD?
We have been working in large complex applications for the last several projects and found some rough edges around both business logic and data access. These edges come from not separating the business concepts effectively in code. Long running business sagas being isolated helped to solve this. We added this advanced support to Highway.Data to help make this easier for us, and those like us that have drank the DDD punch.

##How did you support DDD?
We added a few classes to help support the ability to create domain bounded contexts. These use a generic type parameter to define which domain they contain, and also to segregate the in memory cache for mappings. This allows us to load differing views on similar objects.

###IDomainRepository
This repository has the event interception events, as well as the bounded context.

``` csharp
	public interface IDomainRepository<in T> where T : class
	{
	    event EventHandler<BeforeQuery> BeforeQuery;
	
	    event EventHandler<BeforeScalar> BeforeScalar;
	
	    event EventHandler<BeforeCommand> BeforeCommand;
	
	    event EventHandler<AfterQuery> AfterQuery;
	
	    event EventHandler<AfterScalar> AfterScalar;
	
	    event EventHandler<AfterCommand> AfterCommand;
	
	    IDomainContext<T> DomainContext { get; } 
	}
```
###IDomainContext
This context holds the context level interception events, and is contrained to a `IDomain` type.

```
    public interface IDomainContext<in T> : IDataContext where T : class
    {
        /// <summary>
        ///     The event fired just before the commit of the persistence
        /// </summary>
        event EventHandler<BeforeSave> BeforeSave;

        /// <summary>
        ///     The event fired just after the commit of the persistence
        /// </summary>
        event EventHandler<AfterSave> AfterSave;
    }
```

###IDomain
This is the interface that you implement to define a domain for your objects. It includes any event interceptors, context configurations, connection string, and object mappings.

```
    public interface IDomain
    {
        string ConnectionString { get; }

        IMappingConfiguration Mappings { get;}

        IContextConfiguration Context { get; }

        List<IInterceptor> Events { get; }
    }
```