---
layout: post
title: "Advanced Queries / Commands /Scalars"
date: 2013-10-25 15:11
comments: true
no_homepage: true
order: 32
categories: data feature
---
#Advanced Queries / Commands / Scalars

{% pullquote %}
We have defined standard queries in our [Queries / Commands / Scalars](blog/2013/10/19/queries-slash-commands-slash-scalars/) post, but how to tackle more complex technology specific queries, commands and scalars. These are for the cases where an ORM based operation doesn't make sense (batch insert, bulk delete, complex set based operations). The intent is that you rarely have to use advanced queries, but when you do they work seamlessly. {"Let the technology deal with 95% of the data access, and use advanced queries for the other 5% as needed."} 
{% endpullquote %}

# Example Domain

In all of the examples below, we'll be working with the following business domain, from our Driver's Education company:

```
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

public class DropMake : Command
{
    public DropMake(string make)
    {
        // THIS IS A REALLY BAD WAY TO REMOVE MULTIPLE ROWS
        // IT WOULD NORMALLY BE MUCH BETTER TO USE AN
        // AdvancedCommand TO PERFORM THIS TYPE OF OPERATION
        ContextQuery = c =>
        {
            var cars = c.AsQueryable<Car>().Where(car => car.Make == make);
            foreach (var car in cars)
            {
                c.Remove(car);
            }
        };
    }
}
```

# Advanced Query - When you need a hammer

{% pullquote %}
In the usage of data persistence technologies it is possible to run into corners that cost a significant amount of time to build out of. This has the chance to nullify the speed of development benefits in using the technology. Highway.Data provides a way to step out of this corner by using the base technology of the underlying implementation. {"When you are building something, sometimes you just need a hammer. Highway.Data gives you an easy way to use the hammer, but not affect the utilizing code."} 
{% endpullquote %}

The `AdvancedQuery` is an opt in process for one reason, **it requires that you bind your implementation of the query to the underlying technology.** This is not something to take lightly, but sometimes it is needed. 

```
public class DriversController : Controller
{
    private IRepository repo;

    public DriversController(IRepository repo)
    {
        this.repo = repo;
    }

    public ActionResult RemoveChevy()
    {
        repo.Execute(new DropMake("Chevy"));
        return RedirectToAction("Index", "Home");
    }
}
```
Now that we have a call to the database and out training school has been running for a while we realize that we have 500 "Smith"s in the database. This causes our view to be horrible, and we need to add paging to the call. This is as easy as modifying the usage of `Query` to do the following.

This allows us to reused already defined queries for our paging solution as well. **This is one of the rare cases that you can modify the SQL of a query from outside the query** This will only return the records that are inside the page that you have defined.