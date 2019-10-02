---
layout: post
title: "Simplified Creation Patterns"
date: 2014-01-25 16:45
comments: true
no_homepage: true
categories: [data, feature]
---
#Simplified Creation Patterns

###Repository Factory
We wanted to provide a simple way to construct both domain repositories and simple Repositories. This lead us to create two different factories for repository. We ship a default factory for both of these, but we also ship interfaces for these as a test and extension point.

####Repository Factory
This allows you to construct a simple repository that doesn't need any of the `DomainRepository` features.

```
 	public interface IRepositoryFactory
    {
        /// <summary>
        /// Creates a repository for the requested domain
        /// </summary>
        /// <returns>Domain specific repository</returns>
        IRepository Create();
    }

```

####Domain Repository Factory
This allows you construct a repository specific to any domain that the factory is dependent on.

```
    public interface IDomainRepositoryFactory
    {

        /// <summary>
        /// Creates a repository for the specified <see cref="IDomain"/>
        /// </summary>
        /// <typeparam name="T">Domain for repository</typeparam>
        /// <returns><see cref="IRepository"/></returns>
        IRepository Create<T>() where T : class, IDomain;

        /// <summary>
        /// Creates a repository for the specified <see cref="IDomain"/>
        /// </summary>
        /// <param name="T">Domain for repository</param>
        /// <returns><see cref="IRepository"/></returns>
        IRepository Create(Type type);
    }
```
