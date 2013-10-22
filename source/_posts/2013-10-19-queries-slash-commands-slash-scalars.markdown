---
layout: post
title: "Queries / Commands / Scalars"
date: 2013-10-19 09:21
comments: true
no_homepage: true
order: 30
categories: data feature
---
#Using Queries, Commands and Scalars
The basis for separating concerns in Highway.Data is that the query object itself is the smallest level of data access and can be used to encapsulate the concerns of the "How we get data" from the "What data do I get". In this post will we be diving into the reasoning, implementation, and usage of the different types of Query Objects included with Highway.Data. 

* [Patterns](/blog/2013/10/19/understanding-the-patterns/)
* [Query](#query)
* [Command](#command)
* [Scalar](#scalar)

<a name="query"></a>
#Queries
##Implementation
Lets take a look at the definition of out basic Query interface
``` csharp
public interface IQuery<out T> : IQueryBase
{
    IEnumerable<T> Execute(IDataContext context);
}
```
Execute is the function that our `Repository` will use to execute the query, and it will hand it the context to execute upon. This allows us to create queries without external dependencies, but also allows us to add additional behavior by creating our own context without breaking our queries.

###IEnumerable<T> Vs IQueryable<T>
Notice how the Execution of the query returns an `IEnumerable` instead of an `IQueryable`. This is to basically "seal" the SQL so that additional operations happen in memory. The horrors of lazy loading and LINQ allow us, as developers without realizing it, to load tremendous amounts of data into memory. We have taken steps to make sure that is an intentional choice rather than an unintended consequence.


##Usage
The basic query selects a collection of objects that you tell it to as one operation. If we need to get drivers by last name, consider the following classes and tests.

###Classes Involved
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
Notice how we inherit from the `Query<Driver>` class. This class has most of the base logic for deferred execution and gives us an easy way to define queries. The type parameter of Driver tells us what the query will return. In one line we assign the `ContextQuery`, which is just a delegate to get called when we need the results.

###Tests

For these tests we are using the `InMemoryDataContext` that ships with Highway.Data as a testable stub.

```
[TestMethod]
public void ShouldGetDriversByLastName()
{
	//Arrange
	var context = new InMemoryDataContext();
	context.Add(new Driver(){ LastName = "Liles" });
	context.Add(new Driver(){ LastName = "Rayburn" });

	var query = new DriversByLastName("Liles");

	//Act
	var results = query.Execute(context); 
    
	//Assert
	Assert.AreEqual(1, results.Count());
	Assert.AreEqual("Liles",results.Single().LastName);
}
``` 
###Example Usage with repository
```
IEnumerable<Driver> = repository.Find(new DriversByLastName("Liles"));
```

<a name="command"></a>
#Command
##Implementation
Lets take a look at the definition of out basic Command Interface
``` csharp
public interface ICommand
{
    void Execute(IDataContext context);
}
```
Execute is the function that our Repository will use to execute the command, and it will hand it the context to execute upon. This allows us to create commands without external dependencies, but also allows us to add additional behavior by creating our own context without breaking our commands.

##Usage
The basic command allows us to fire off a set of operations without a return against our persistence store. This command could be updates, deletes, or even firing SQL Jobs. Lets consider the classes and tests involved

###Classes Involved
```
public class Driver 
{ 
	public int Score { get; set; }
	public string LastName { get; set; } 
}

public class SetDriverScoreByLastName : Command
{
    public SetDriverScoreByLastName(string lastName, int score)
    {
        ContextQuery = context => 
 		{
			var drivers = context.AsQueryable<Driver>().Where(e => e.LastName == lastName);
			foreach(var driver in drivers)
			{
				driver.Score = score;
			}
			context.Commit();
		}
    }
}
```
Notice how we inherit from the `Command` class. This class has most of the base logic for deferred execution and gives us an easy way to define commands. We can set the `ContextQuery` to be a multiple line delegate like above, or a single operation.

>###***Commands are not deferred, they execute immediately***

###Tests

For these tests we are using the `InMemoryDataContext` that ships with Highway.Data as a testable stub.

```
[TestMethod]
public void ShouldSetScoresByLastName()
{
	//Arrange
	var context = new InMemoryDataContext();
	context.Add(new Driver(){ LastName = "Liles" });
	context.Add(new Driver(){ LastName = "Rayburn" });

	var command = new SetDriverScoreByLastName("Liles", 100);

	//Act
	command.Execute(context); 
    
	//Assert
	var result = context.AsQueryable<Driver>().Single(x => x.LastName == "Liles"); 
	Assert.AreEqual("Liles", results.LastName);
	Assert.AreEqual(100, results.Score);
}
```
###Example Usage with repository
```
repository.Execute(new SetDriverScoreByLastName("Liles", 100));
```


<a name="scalar"></a>
#Scalar
##Implementation
Lets take a look at the definition of out basic Scalar interface
``` csharp
public interface IScalar<out T>
{
    T Execute(IDataContext context);
}
```
##Usage
The basic scalar selects a single object that you tell it to as one operation. If we need to get a driver by last name, consider the following classes and tests. we can also return out non entity types such as a count of drivers by score;

###Classes Involved
```
public class Driver 
{ 
	public int Score { get; set; }
	public string LastName { get; set; } 
}

public class FirstDriverByLastName : Scalar<Driver>
{
    public FirstDriverByLastName(string lastName)
    {
        ContextQuery = context => context.AsQueryable<Driver>().FirstOrDefault(e => e.LastName == lastName);
    }
}

public class PassingDrivers : Scalar<int>
{
    public PassingDrivers()
    {
        ContextQuery = context => context.AsQueryable<Driver>().Count(x => x.Score > 75);
    }
}
```
Notice how we inherit from the `Scalar<Driver>` or `Scalar<int>` class. The type parameter defines the type we are returning with the scalar, and the base class gives us all the logic to execute and log the information about the execution. 

>###***Scalars are not deferred, they execute immediately***


###Tests

For these tests we are using the `InMemoryDataContext` that ships with Highway.Data as a testable stub.

```
[TestMethod]
public void ShouldGetFirstDriverByLastName()
{
	//Arrange
	var context = new InMemoryDataContext();
	context.Add(new Driver(){ LastName = "Liles" });
	context.Add(new Driver(){ LastName = "Rayburn" });

	var query = new FirstDriverByLastName("Liles");

	//Act
	var result = query.Execute(context); 
    
	//Assert
	Assert.AreEqual("Liles",result.LastName);
}

[TestMethod]
public void ShouldGetCountOfPassingDrivers()
{
	//Arrange
	var context = new InMemoryDataContext();
	context.Add(new Driver(){ LastName = "Liles", Score = 50 });
	context.Add(new Driver(){ LastName = "Rayburn", Score = 100 });

	var query = new PassingDrivers();

	//Act
	var result = query.Execute(context); 
    
	//Assert
	Assert.AreEqual(1,result);
}
```

###Example Usage with repository
```
Driver driver = repository.Find(new DriverByLastName("Liles"));
int passingDrivers = repository.Find(new PassingDrivers());
``` 