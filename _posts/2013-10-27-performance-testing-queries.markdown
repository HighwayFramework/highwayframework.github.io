---
layout: post
title: "Performance Testing Queries"
date: 2013-10-27 16:31
comments: true
no_homepage: true
order: 34
categories: [data, feature]
---

When dealing with data access you should always adhere to Jackson's rules of optimization. For convenience they are right here:
###M.A. Jackson (Principles of Program Design, 1975) wrote:
**Rule 1.** Don't do it

**Rule 2. (for experts only)** Don't do it yet.

Performance optimization should only be done when needed, but when you need to it should be easy to execute and to measure. With this in mind, Highway took some of the heavy lifting out of the hands of the developers. This is normally the tools that we will use to identify a query that is not performing up to par so we can change it. Let's take the previous example of deleting all cars of a certain make.

# Example Domain

In all of the examples below, we'll be working with the following business domain, from our Driver's Education company:

``` csharp
public class Car
{
    public int Id { get; set; }
    public string Make { get; set; }
    public string Model { get; set; }
    public string Year { get; set; }
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

# Performance Tests
## These should *ideally* run against production like data

I am going to use a command for delete, but this could be just as easily done with a `Scalar` or `Query`.

We want to measure the time it takes to execute the query and measure it against our maximum. If you cannot define a specific maximum for the test, you should look at Jackson rule 1. If your maximum doesn't fail this test, look at Jackson rule 2.

```
[TestMethod]
public void ShouldDeleteInUnder250Milliseconds()
{
    var context = new DataContext(Settings.Default.Connection, new DataMappings());

    var dropMake = new DropMake("Chevy");

    dropMake.RunPerformanceTest(context,false,250);
}
```

If the time exceeds maximum the performance test will throw an exception which will fail the test. The time and expected are in the exception.

Let's take a closer look at this line
```
dropMake.RunPerformanceTest(context,false,250);
```
The 1st parameter, context, is the data connection we use to execute the test.

The 2nd parameter, false, is a flag telling us to include the start up time of the context in the total time. This is useful with ORMs like Entity Framework or NHibernate that have a large one time cost on start up. We give you the option of excluding that time from your evaluation.

The 3rd parameter, 250, is the total milliseconds allowed for the execution. If it exceeds this the test fails.
