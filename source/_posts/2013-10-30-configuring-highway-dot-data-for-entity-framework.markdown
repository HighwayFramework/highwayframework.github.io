---
layout: post
title: "Entity Framework - Configuring Highway.Data"
date: 2013-10-30 20:20
comments: true
no_homepage: true
order: 101
categories: data feature
---
##Getting Started
The first step to getting Highway.Data running on Entity Framework is to install both Entity Framework and Highway.Data.EntityFramework with the below command.

``` plain
Install-Package Highway.Data.EntityFramework
```

This will bring the install down and put it in our project. 

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
```

##Database to Entity Mappings
Now we need to create our database mappings. Highway.Data doesn't redefine the mapping syntax, it just makes them injectable into the `DataContext`. To do this we defined an interface `IMappingConfiguration` for you to implement that will allow us to inject your domain into a pre-built `DataContext`. 
**The best practice is to name this class after the aggregate root in your domain, so ours is DriverExams**

``` csharp
public class DriversExams : IMappingConfiguration
{
    public void ConfigureModelBuilder(DbModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Driver>(); //This is the inline/fluent config

        modelBuilder.Configurations.Add(new DriverMap()); //This is the class based config
    }
}

public class DriverMap : EntityTypeConfiguration<Driver>
{
    public DriverMap()
    {
        this.ToTable("Drivers");
		//You can do anything here that EF supports
    }
}
```

That is all it takes to get our Database schema mapped to our entities. *As an aside, EF powertools will reference engineer the `EntityTypeConfiguration<T>` classes for you, and then you can just add them to your `IMappingConfiguration`*

##Context Level Configuration
We setup a `DefaultContextConfiguration` by default, but if you disagree with our opinions about lazy loading etc.. you can change that. You just need to implement a class for `IContextConfiguration` like below.

``` csharp
public class DefaultContextConfiguration : IContextConfiguration
{
    public void ConfigureContext(DbContext context)
    {
        context.Configuration.LazyLoadingEnabled = false;
        context.Configuration.ProxyCreationEnabled = false;
		//Here you can do any context level configuration changes that EF supports
    }
}
```

This will let you change fundemental behavior of the context.

##Logging Configuration

Then you need to send in a logger, but those details are covered in our [Logging Post](/blog/2013/10/28/logging-with-datacontext/) because it is not Entity Framework specific.

##Using it all
Last but not least we need to use our configured pieces like so:

``` csharp
var context = new DataContext("Your connection string here", new DriversExams(), new DefaultContextConfiguration(), new NoOpLogger());
```

We normally do this via our favorite IoC Container, but alas that is another guide.