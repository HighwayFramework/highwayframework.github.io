---
layout: post
title: "Entity Framework - Testable Adapter for DbContext"
date: 2013-10-15 12:58
comments: true
order: 100
categories: data feature
---

# Installing Highway.Data.EntityFramework

Our first and most important feature of Highway.Data.EntityFramework is wrapping the Highway.Data `IDataContext` interface around the `DbContext` of Entity Framework.  You get started with Highway.Data.EntityFramework by simply opening the Package Manager Console and typeing:

``` powershell
Install-Package Highway.Data.EntityFramework
```

This will bring in Entity Framework, Highway.Data, Highway.Pavement, and some common wrappers around Logging and Service location.  

# Introducing the IDataContext

Now, let's create a variable of type `IDataContext` and provide it an instance of the class `DataContext`, like so:

``` csharp
IDataContext context = new DataContext("server=.;database=master;integrated security=true;");
```

Note, we provide our connection string to the `DataContext` class when it is instantiated.  Let's look at the definition of `IDataContext`:

```
public interface IDataContext : IDisposable
{
    IEventManager EventManager { get; set; }

    T Add<T>(T item) where T : class;
    IQueryable<T> AsQueryable<T>() where T : class;
    int Commit();
    T Reload<T>(T item) where T : class;
    T Remove<T>(T item) where T : class;
    T Update<T>(T item) where T : class;
}
```

The interface provides a way to do all of the CRUD operations:

* **C**reate - `Add<T>(T item)`
* **R**ead - `AsQueryable<T>()`
* **U**pdate - `Update<T>(T item)`
* **D**elete - `Remove<T>(T item)`

In addition we've provided a way to do two other important things:

* Refresh an object from the Database via `Reload<T>(T item)`
* Commit all work as a single transaction via `Commit()`

## DbContext and Entity Framework

Inside the Highway.Data.EntityFramework version of `DataContext`, it is worth of note that our `DataContext` class **is** an instance of `DbContext`.  Our declaration looks like so:

```
public class DataContext : DbContext, IEntityDataContext, IObservableDataContext, IDataContext, IDisposable
```
Anytime you have existing code which requires a DbContext, you can instead provide an instance of the DataContext class.

# Testing with DataContext

From a testing perspective this provides great abstractions.  Consider the following classes and test:

```
public class Driver { public string Name { get; set; } }

public class DriverEducationService
{
    private IDataContext context;
    public DriverEducationService(IDataContext context)
    {
        this.context = context;
    }

    public Driver GetDriver(string name)
    {
        return context.AsQueryable<Driver>().First(e => e.Name == name);
    }
}
```

A simple class that retrieves driver by name. **Please note, we are discussing only `IDataContext` here, we normally recommend querying via `IRepository` in Highway.Data**

Now let's write a test that ensures that works, without having to touch a database:

```
[TestClass]
public class DriverEducationServiceTests
{
    [TestMethod]
    public void GetDriver_ShouldRetrieveADriverByName()
    {
        // Arrange
        var context = new InMemoryDataContext();
        context.Add(new Driver { Name = "Devlin Liles" });
        var target = new DriverEducationService(context);

        // Act
        var driver = target.GetDriver("Devlin Liles");

        // Assert
        Assert.AreEqual("Devlin Liles", driver.Name);
    }
}
```

Now, this test doesn't use mocking because we provide an `InMemoryDataContext` with Highway.Data which removes the need for it in most cases.  But when it doesn't remove that need, we can also re-write the same test using a mocking framework like Rhino.Mocks very simply:

```
[TestClass]
public class DriverEducationServiceTests_WithMocking
{
    [TestMethod]
    public void GetDriver_ShouldRetrieveADriverByName()
    {
        // Arrange
        var context = MockRepository.GenerateMock<IDataContext>();
        context.Expect(e => e.AsQueryable<Driver>())
            .Return(new List<Driver> 
            { 
                new Driver { Name = "Devlin Liles" } 
            }.AsQueryable());
        var target = new DriverEducationService(context);

        // Act
        var driver = target.GetDriver("Devlin Liles");

        // Assert
        Assert.AreEqual("Devlin Liles", driver.Name);
    }
}
```
