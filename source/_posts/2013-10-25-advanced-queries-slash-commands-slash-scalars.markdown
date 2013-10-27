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


All of the `AdvancedQuery`, `AdvancedScalar`, and `AdvancedCommand` are an opt in process for one reason, **it requires that you bind your implementation of the query to the underlying technology.** This is not something to take lightly, but sometimes it is needed. 

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
    public int Score { get; set; }
    public Instructor Instructor { get; set; }
}

public class Car
{
    public int Id { get; set; }
    public string Make { get; set; }
    public string Model { get; set; }
    public string Year { get; set; }
}

public class Top5PercentileOfDrivers : Query<Driver>
{
    public Top5PercentileOfDrivers()
    {
        ContextQuery = context =>
        {
            var scores = context.AsQueryable<Driver>().Select(x => x.Score);
            int percentileScore =
                Convert.ToInt32(Math.Round((5/100)*scores.Count() + 0.5, MidpointRounding.AwayFromZero));
            return context.AsQueryable<Driver>().OrderByDescending(x => x.Score).Take(percentileScore);
        };
    }
}

public class SwapInstructors : Scalar<int>
{
    public SwapInstructores(Instructor currentInstructor, Instructor newInstructor)
    {
        ContextQuery = context =>
        {
            foreach(var driver in currentInstructor.Drivers)
			{
				driver.Instructor = newInstructor;
			}
			context.Commit();
        };
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

# Advanced Query - Sometimes you need a Stored Procedure
In the case where we want to do large set based calculation it makes sense to use the power of the underlying persistence engine. Databases have been designed for set based operations, and here is where the power of stored procedures or parameterized SQL comes in. We are going to use a stored procedure to return the top 5 percentile of drivers that have used our training service. The advanced version of this would be below.

```
public class Top5PercentileOfDrivers : AdvancedQuery<Driver>
{
    public Top5PercentileOfDrivers()
    {
        ContextQuery = context => context.ExecuteSqlQuery<Driver>("exec topPercentileDrivers @percentile", new SqlParameter("percentile",5));
    }
}
```

# Advanced Commands -  When you need a hammer
{% pullquote %}
In the usage of data persistence technologies it is possible to run into corners that cost a significant amount of time to build out of. This has the chance to nullify the speed of development benefits in using the technology. Highway.Data provides a way to step out of this corner by using the base technology of the underlying implementation. When you are building something, sometimes you just need a hammer. {"Highway.Data gives you an easy way to use the hammer, but doesn't require every problem to be a nail."} 
{% endpullquote %}
```
public class DropMake : AdvancedCommand
{
    public DropMake(string make)
    {
        ContextQuery = c => c.ExecuteSqlCommand("DELETE FROM Cars WHERE Make = @make",new DbParameter[] {new SqlParameter("make", make)});
    }
}
```
#Advanced Scalar
In the instance that we need to make a lot of changes but also return some value from the database, we can use an `AdvancedScalar`. 
```
public class SwapInstructores : AdvancedScalar<int>
{
    public SwapInstructores(Instructor currentInstructor, Instructor newInstructor)
    {
        DbParameter[] parameters = new DbParameter[]
        {
            new SqlParameter("old", currentInstructor.Id),
            new SqlParameter("new", newInstructor.Id), 
        };
        ContextQuery = c => c.ExecuteSqlCommand("UPDATE DRIVERS SET InstructorId = @new WHERE InstructorId = @old", parameters);
    }
}
```

Each of these examples gives one of the many usages of `AdvancedQuery`/`AdvancedCommand`/`AdvancedScalar`, but when you need the underlying provider this is your route. When used carefully this allows us to serve both the the needs of our application, but also the needs of our data storage.