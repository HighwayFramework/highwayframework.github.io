---
layout: post
title: "Testable Data Access At Every Layer"
date: 2013-10-15 12:58
comments: true
order: 100
categories: data feature
---

# Installing Highway.Data

Our first and most important feature of Highway.Data is wrapping the Highway.Data `IDataContext` interface around the specific ( normally leaky ) implementation of the persistence technology.  You get started with Highway.Data by simply opening the Package Manager Console and typeing which ever of our adapters you decide to use:

``` powershell
Install-Package Highway.Data.EntityFramework
Install-Package Highway.Data.NHibernate
Install-Package Highway.Data.RavenDb
```

This will bring in your chosen persistence framework, Highway.Data, Highway.Pavement, and some common wrappers around Logging and Service location.  

#Mocking Framework
Chose any one you like.

1. [Rhino Mocks](http://hibernatingrhinos.com/oss/rhino-mocks) We use this in developing the Highway Framework, so this will be what we use in examples.
2. [Moq](http://www.moqthis.com/)
3. [Microsoft Fakes](http://msdn.microsoft.com/en-us/library/hh549175%28v=vs.120%29.aspx)
4. [JustMock](http://www.telerik.com/products/mocking.aspx)
5. [EasyMock.NET](http://sourceforge.net/projects/easymocknet/)
6. [TypeMock](http://www.typemock.com/)
7. [NSubstitute](http://nsubstitute.github.io/)
8. [NMock](http://nmock3.codeplex.com/)
9. [FakeItEasy](https://github.com/fakeiteasy)
10. the other dozens we didn't enumerate

#Introducing the IRepository
Let's create a variable of type `IRepository` and provide it a mock, like so:

``` csharp
IRepository repository = MockRepository.GenerateMock<IRepository>();
```

Now, Let's look at the definition of `IRepository`:

```
public interface IRepository
{
    IDataContext Context { get; }
    IEventManager EventManager { get; }
    IEnumerable<T> Find<T>(IQuery<T> query);
    T Find<T>(IScalar<T> query);
    void Execute(ICommand command);
}
```
The interface provides a way to do Query Object based execution for Queries / Commands / Scalars.If you would like to know more about how these are wired together at a fundemental level, check out our [Patterns Post](/blog/2013/10/19/understanding-the-patterns/).

* Unit of Work - `Context`
* Event/Interceptor Management - `EventManager`
* Data Access - `Find<T>(IQuery<T> query) && Find<T>(IScalar<T> query)`
* Data Execution - `Execute(ICommand command)`

From a testing perspective this provides great abstractions. We can test our business logic without having to worry about even the structure of our queries.  Consider the following classes and test:

```
public class Driver { public string Name { get; set; } }

public class DriverEducationService
{
    private IRepository repository;
    public DriverEducationService(IRepository repository)
    {
        this.repository = repository;
    }

    public Driver GetDriver(string name)
    {
        return repository.Find(new DriverByName(name));
    }
}
```
A simple class that retrieves driver by name. This query is just the way we have codified that query into something that would make sense for logic to know. My logic should know it needs to find a driver by name, but shouldn't know how to do that. This gives us that testable abstraction.

Now let's write a test that ensures that works, without having to touch a database:

```
[TestClass]
public class DriverEducationServiceTests
{
    [TestMethod]
    public void GetDriver_ShouldRetrieveADriverByName()
    {
        // Arrange
        IRepository repository = MockRepository.GenerateMock<IRepository>();
        repository.Expect(x => x.Find(Arg<DriverByName>.Is.NotNull)).Return(new Driver("Devlin Liles"));
        var target = new DriverEducationService(repository);

        // Act
        var driver = target.GetDriver("Devlin Liles");

        // Assert
        Assert.AreEqual("Devlin Liles", driver.Name);
    }
}
```

Now this lets us test up, but we still need to test our selection criteria in the query. Aka, test down. I would like to do this without a database. Let's give it a shot.

# Introducing the IDataContext
Now, let's create a variable of type `IDataContext` and provide it an instance of the class `DataContext`, like so:

``` csharp
IDataContext context = MockRepository.GenerateMock<IDataContext>();
```

Now, Let's look at the definition of `IDataContext`:

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

## Framework Specifics

###Entity Framework - DbContext
Inside the Highway.Data.EntityFramework version of `DataContext`, it is worth of note that our `DataContext` class **is** an instance of `DbContext`.  Our declaration looks like so:

```
public class DataContext : DbContext, IEntityDataContext, IObservableDataContext, IDataContext, IDisposable
```
Anytime you have existing code which requires a DbContext, you can instead provide an instance of the DataContext class.

###NHibernate - ISession
Inside the Highway.Data.NHibernate version of `DataContext`, it is worth of note that our `DataContext` class **is** an implementor of `ISession`.  Our declaration looks like so:

```
public class DataContext : ISession, IObservableDataContext, IDisposable
```
Anytime you have existing code which requires a `ISession`, you can instead provide an instance of the DataContext class.

###RavenDb - IDocumentSession
Inside the Highway.Data.RavenDb version of `DataContext`, it is worth of note that our `DataContext` class **is** an implementor of `IDocumentSession`.  Our declaration looks like so:

```
public class DataContext : IDocumentSession, IObservableDataContext, IDisposable
```
Anytime you have existing code which requires a `IDocumentSession`, you can instead provide an instance of the DataContext class.


# Testing with DataContext

From a testing perspective this provides great abstractions.  Consider the following classes and test:

```
public class Driver { public string Name { get; set; } }

public class DriverByName : Scalar<Driver>
{
	public DriverByName(string name)
	{
		ContextQuery = context => context.AsQueryable<T>().SingleOrDefault(x => x.Name == name);
	}
}
```

A simple class that retrieves driver by name. 

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
		context.Add(new Driver { Name = "Tim Rayburn" });
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
public class DriverEducationServiceTests
{
    [TestMethod]
    public void GetDriver_ShouldRetrieveADriverByName()
    {
        // Arrange
        var context = MockRepository.GenerateMock<IDataContext>();
		context.Expect(x => x.AsQueryable<Driver>())
			.Return(new List<Driver>()
				{
					new Driver { Name = "Devlin Liles" }, 
					new Driver { Name = "Tim Rayburn" }
				}.AsQueryable());
        
        var target = new DriverEducationService(context);

        // Act
        var driver = target.GetDriver("Devlin Liles");

        // Assert
        Assert.AreEqual("Devlin Liles", driver.Name);
    }
}
```
This test uses the mock of IDataContext to provide a way for testing filter and query logic without having a database handy. It uses a great LINQ method `AsQueryable()` to help with the return. It is a bit heavier for normal queries.

#InMemoryDataContext
Just a few points to keep in mind with `InMemoryDataContext`.

1. It does not now, nor will it ever support AdvancedQuery, AdvancedCommand, or AdvancedScalar.
2. It modifies objects by ref in real time, so Commit will only rescan the graph for added or removed objects. **This means changing a primitive property will change it even if you don't call commit**
