---
layout: post
title: "Queries / Commands / Scalars"
date: 2013-10-19 09:21
comments: true
no_homepage: true
categories: data onramp-mvc onramp-services pavement roadcrew onramper insurance release feature
---

The basis for separating concerns in Highway.Data is that the query itself is the smallest level of data access and can be used to encapsulate the concerns of the "How we get data" from the "What data do I get". Lets first look at the two kinds of query object we have.

``` csharp
public interface IQuery<out T> : IQueryBase
{
    IEnumerable<T> Execute(IDataContext context);
}
```

###IEnumerable<T> Vs IQueryable<T>
Notice how the Execution of the query returns an IEnumerable instead of an IQueryable. This is to basically "seal" the SQL so that additional operations happen in memory. The horrors of lazy loading and LINQ allow us, as developers without realizing it, to load tremendous amounts of data into memory. We have taken steps to make sure that is an intentional choice rather than an unintended consequence.


### Usage
The basic query selects a collection of objects that you tell it to as one operation. If we need to get drivers by last name, consider the following classes and tests.

```
public class Driver { public string LastName { get; set; } }

public class DriversByLastName : Query<Driver>
{
    public DriversByLastName(string lastName)
    {
        ContextQuery = context => context.AsQueryable<Driver>().Where(e => e.LastName == lastName);
    }
}
```

Notice how we inherit from the `Query<Driver>` class. This class has most of the base logic for deferred execution and gives us an easy way to define queries. The type parameter of Driver tells us what the query will return. In one line we assign the ContextQuery, which is just a delegate to get called when we need it. 

