---
layout: post
title: "Pagination Of Queries"
date: 2013-10-25 14:11
comments: true
no_homepage: true
order: 31
categories: data feature
---
#Paging Queries

{% pullquote %}
We have gone through how to define and use Queries in our [Queries / Commands / Scalars](blog/2013/10/19/queries-slash-commands-slash-scalars/) post, but now lets talk about value adds on those Queries. If you want to page an existing `Query<T>`, we believe that you should be able to quickly and easily.{"The purpose of a query is reuse and test-ability, and if you cannot reuse it why do it?"} 
{% endpullquote %}

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

public class DriversByLastName : Query<Driver>
{
    public DriversByLastName(string lastName)
    {
        ContextQuery = context => context.AsQueryable<Driver>().Where(e => e.LastName == lastName);
    }
}
```

# Paging The IQuery - LINQ style

This method we chose for paging is one that should be familiar to the entire .NET community at this point. We are using `.Skip(int numberOfObjects)` and `.Take(int numberOfObjects)`. Take the following controller method to get drivers by name

```
public class DriversController : Controller
{
    private IRepository repo;

    public DriversController(IRepository repo)
    {
        this.repo = repo;
    }

    public ActionResult ByLastName(string lastName)
    {
        IEnumerable<Driver> drivers = repo.Find(new DriversByLastName(lastName));
        return View(drivers);
    }
}
```
Now that we have a call to the database and out training school has been running for a while we realize that we have 500 "Smith"s in the database. This causes our view to be horrible, and we need to add paging to the call. This is as easy as modifying the usage of `Query` to do the following.

```
public class DriversController : Controller
{
    private IRepository repo;

    public DriversController(IRepository repo)
    {
        this.repo = repo;
    }

    public ActionResult ByLastName(string lastName, int page)
    {
		var pageSize = 10;
        IEnumerable<Driver> drivers = repo.Find(new DriversByLastName(lastName)
												    .Skip(page * pageSize).Take(pageSize));
        return View(drivers);
    }
}
```
This allows us to reused already defined queries for our paging solution as well. **This is one of the rare cases that you can modify the SQL of a query from outside the query** This will only return the records that are inside the page that you have defined.