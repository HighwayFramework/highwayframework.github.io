---
layout: post
title: "Easy RESTful ApiControllers"
date: 2014-01-25 19:09
comments: true
no_homepage: true
order: 30
categories: onramp-mvc feature
---

Using the Domain concepts and Prebuilt Queries of Highway.Data, we have included a basic RESTful base class for your ApiControllers.  This feature is very easy to use, simply inherit from `BaseRestApiController`.  That base class has two constructor requirements:

1. `IDomainRepositoryFactory` this class comes from Highway.Data and is already registered with our IoC.
2. `RestfulOperations` this is a **FLAGS** enumeration that lets you control how the base class behaves.

An implementation controller would look like this:

``` csharp
public class DriverController : BaseRestApiController<Domain, Driver, Guid>
{
	public DriverController(IDomainRepositoryFactory factory) :
		base(factory, RestfulOperations.All) { }
		
	public override CopyEntityValues(Driver source, Driver destination)
	{
		// copy values from one instance to another here.
	}
}
```

This class would support all five of our pre-built functions, because it passed `RestfulOperations.All`, these are:

* GetAll - This is the default get with no `id` and returns ALL Drivers
* GetOne - This returns one driver, by `id`
* Put - Accepts a `PUT` verb and updates a singular Driver.  Uses `CopyEntityValues` above to control what can be updated.
* Delete - Deletes a Driver by `id`
* Post - Accepts a `POST` verb and inserts a singular Driver.

Importantly this enums is **FLAGS** based, and as such the values can be OR'ed together.  For instance, if you wanted to just support GetOne and Post you could change the line above to:

```
	public DriverController(IDomainRepositoryFactory factory) :
		base(factory, RestfulOperations.GetOne | RestfulOperations.Post) { }
```

There are some logical pre-built combinations already supported in the enum, which is defined as follows:

```
        [Flags]
        public enum RestOperations
        {
            GetAll = 1,
            GetOne = 2,
            Post = 4,
            Put = 8,
            Delete = 16,
            ReadOnly = 3,
            WriteOnly = 28,
            All = 31
        }
```