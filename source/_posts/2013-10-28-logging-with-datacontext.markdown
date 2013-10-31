---
layout: post
title: "Logging"
date: 2013-10-28 17:59
comments: true
no_homepage: true
order: 21
categories: data feature
---
We all have written/supported an application that had zero logging. It is like playing two sided blindfolded chess in the dark. We didn't want to pass that on to the users of Highway so logging is backed right into the toolset. We rely on [Common.Logging 2.1.1.0]() for our logging API, because this allows you, our users, to use your favorite logging facility. I am going to use a simple console logger for these examples, but it could be any Common.Logging adapter.

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
```

#Adding and Removing Items
When you add an item there are several things that could/should happen. Knowing at which point the failure happens is important. This is where Highway Framework makes things easier on you. We want to know where the operations are executing and how the context is operating on a configurable level. This is going to use the standard log levels to get output from the system. Consider the following Test.

```
[TestMethod]
public void ShouldLogAtDebugLevel()
{
    //arrange 
    var logger = new ConsoleOutLogger("Testing", LogLevel.Debug, true, true, true, @"dd/mm/yyyy hh:mm:ss");
    var target = new DataContext(Settings.Default.Connection, new DriversEducationMappings(), logger);

    //act
    target.Add(new Driver("Devlin", "Liles"));
    target.Add(new Driver("Tim", "Rayburn"));
    target.Add(new Driver("Jay", "Smith"));
    target.Add(new Driver("Brian", "Sullivan"));
    target.Add(new Driver("Cori", "Drew"));

    target.Commit();

    foreach (var driver in target.AsQueryable<Driver>())
    {
        target.Remove(driver);
    }

    target.Commit();

    //assert
    Assert.Inconclusive("We fail here to get the output from console nice and easy");

}
```
The output from this test is below

	30/40/2013 05:40:07 [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - 	Commited 5 Changes
	30/40/2013 05:40:07 [DEBUG] Testing - Querying Object Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Queried Object Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	30/40/2013 05:40:07 [DEBUG] Testing - 	Commited 5 Changes

You can see that you get the step by step of what the application is doing on Debug but if you want even more information, you can up the game with LogLevel.Trace - This will trace every action start and finish.

	10/30/2013 5:49:10 PM [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	10/30/2013 5:49:10 PM [TRACE] Testing - Added Object Highway.DriversEducation.GettingStarted.Driver
	...
	10/30/2013 5:49:10 PM [TRACE] Testing - 	Commit
	10/30/2013 5:49:10 PM [DEBUG] Testing - 	Commited 5 Changes
	10/30/2013 5:49:10 PM [DEBUG] Testing - Querying Object Driver
	10/30/2013 5:49:10 PM [DEBUG] Testing - Queried Object Driver
	10/30/2013 5:49:10 PM [DEBUG] Testing - Removing Object Highway.DriversEducation.GettingStarted.Driver
	10/30/2013 5:49:10 PM [TRACE] Testing - Removed Object 
	...
	10/30/2013 5:49:10 PM [TRACE] Testing - 	Commit
	10/30/2013 5:49:10 PM [DEBUG] Testing - 	Commited 5 Changes

This log level also lets you see the guts of when the model binding hits are being taken.

	10/30/2013 5:51:47 PM [DEBUG] Testing - Adding Object Highway.DriversEducation.GettingStarted.Driver
	10/30/2013 5:51:47 PM [DEBUG] Testing - 	OnModelCreating
	10/30/2013 5:51:47 PM [TRACE] Testing - 		Mapping : DriversEducationMappings

Or Even when we execute a function/stored procedure

	10/30/2013 5:54:16 PM [TRACE] Testing - Executing SQL Select * from Drivers Where LastName = @lastName, with parameters lastName : Liles : String	

If you have code that is reloading objects to refresh them from the database using `Reload<T>(T item)` then you would see something like this.
	
	10/30/2013 5:56:49 PM [TRACE] Testing - Retrieving State Entry For Object Highway.DriversEducation.GettingStarted.Driver
	10/30/2013 5:56:49 PM [DEBUG] Testing - Reloading Object Highway.DriversEducation.GettingStarted.Driver
	10/30/2013 5:56:49 PM [TRACE] Testing - Reloaded Object Highway.DriversEducation.GettingStarted.Driver
	
Keep an eye out because in the next version we will be introducing the following logging features.
	
1. Repository Level Logging of Queries/Commands/Scalars
2. Non-Debugging Levels for always on health logging of the DataContext
3. Performance Logging on Trace Level for Commits and Repository Level Items