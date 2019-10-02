---
layout: post
title: "What should I register with my IoC container?"
date: 2013-11-07 15:09
comments: true
no_homepage: true
categories: [data, faq]
---

If you are going to use an IoC container, regardless of which one, here is what needs to be registered, and some thoughts about object lifetime.  If we do not specify a lifetime, you can assume that singleton is acceptable.

## Highway.Data

* `IRepository` should resolve to `Repository` on either a transient (new every request) or per web request (if you're in a website) object lifetime.
* `IEventManager` should resolve to `EventManager`
* `ILog` should resolve to either a [Common.Logging] implementation for your chosen logger, or an instance of `NoOpLogger` from [Common.Logging].

## Highway.Data.EntityFramework

* `IDataContext` should resolve to `DataContext` on either a transient (new every request).
  * Your IoC will need to inject a connection string, named `connectionString` to the constructor, in whatever way such primitive dependencies are handled by your container.
* `IMappingConfiguration` should resolve to a type you have created that implements this interface and maps all of your entities.
* `IContextConfiguration` should resolve to `DefaultContextConfiguration` **or** a class you've created should you need to change how the Context is configured from our defaults.

## Highway.Data.NHibernate

* `IDataContext` should resolve to `DataContext` on either a transient (new every request).
* `ISession` should resolve to a call to your configured `ISessionFactory`.  These are standard NHibernate interfaces, and you should follow their guidance regarding object lifetime.

## Highway.Data.RavenDb
* `IDataContext` should resolve to `DataContext` on either a transient (new every request).
* `IDocumentSession` should resolve to a call to your configured `IDocumentStore`.  These are standard RavenDb interfaces, and you should follow their guidance regarding object lifetime.

[Common.Logging]:		http://netcommon.sourceforge.net/
