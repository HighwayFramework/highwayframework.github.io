---
layout: post
title: "Eager Loading with Queries and Scalars"
date: 2013-10-28 17:38
comments: true
no_homepage: true
order: 35
categories: data feature
---
When we define a query, sometimes we need to load a graph of related objects as well. This can be impressively helpful with an ORM, but it is also more expensive per query. We have to balance the cost of the query graph with the ease of loading related objects in the code base. The approach to this varies by the underlying ORM that you are using, so please click the link for the section that pertains to you.

[Entity Framework](#ef)

[NHibernate](#nhibernate)

# Example Domain

In all of the examples below, we'll be working with the following business domain, from our Driver's Education company:

``` csharp
public class Instructor
{
    public int Id { get; set; }
    public ICollection<Driver> Drivers { get; set; }
}

public class Driver
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public Car Car { get; set; }
}

public class Car
{
    public int Id { get; set; }
    public string Make { get; set; }
    public string Model { get; set; }
    public string Year { get; set; }
}

public class InstructorById : Scalar<Instructor>
{
    public InstructorById(int id)
    {
        ContextQuery = context => context.AsQueryable<Instructor>()
										 .SingleOrDefault(e => e.Id == id);
    }
}
```

##Use Case
Let's assume that we now need to load Drivers and Cars for the instructor. Because we are only loading one Instructor this should be fairly light weight on the database.

<a name="ef"></a>
#Entity Framework
Entity Framework got the API for this correct, so when you unit test this without a database it does nothing with the include call. This allows us to not need an advanced query for the Entity Framework version of the query.

```
public class InstructorById : Scalar<Instructor>
{
    public InstructorById(int id)
    {
        ContextQuery = context => context.AsQueryable<Instructor>()
									.Include(x => x.Drivers.Select(c => c.Car))
									.SingleOrDefault(e => e.Id == id);
    }
}
```
Notice to traverse a collection we have to use the `Select` method, this is a quark of LINQ and Entity Framework. As you see below in the NHibernate solution there is a slightly more elegant way they could have done this, but it comes with a hefty cost.

<a name="nhibernate"></a>
#NHibernate
In the NHibernate usage we will have to bind ourselves to their specific API, which means that we will have to use an `AdvancedQuery`. 
```
public class InstructorById : AdvancedScalar<Instructor>
{
    public InstructorById(int id)
    {
        ContextQuery = context => context.Query<Instructor>()
									.FetchMany(x => x.Drivers)	
								    .ThenFetch(x => x.Car)
									.SingleOrDefault(e => e.Id == id);
    }
}
```
This reads more fluently but causes us to be bound to NHibernate. **It also has one massive bug in NHibernate's LINQ provider.** In the above query you will only get one related driver and one related car. This is because it applies a Top 1 to the queries. To avoid this you must do the below query.
```
public class InstructorById : AdvancedScalar<Instructor>
{
    public InstructorById(int id)
    {
        ContextQuery = context => context.Query<Instructor>()
									.FetchMany(x => x.Drivers)	
								    .ThenFetch(x => x.Car)
									.Where(e => e.Id == id)
									.ToList() // This forces execution without the Top 1 of a single or default
									.SingleOrDefault();
    }
}
```
**This is a bug in NHibernate**