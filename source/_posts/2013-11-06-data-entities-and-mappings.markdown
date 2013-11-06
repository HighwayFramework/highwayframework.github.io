---
layout: post
title: "Data - Entities and Mappings"
date: 2013-11-06 10:34
comments: true
order: 100
categories: onramp-mvc feature
---

As part of Highway.OnRamp.MVC.Data we include several classes which help you in creating Entity Framework entities and mappings.  These types are not required, but they are meant to assist you.

## Base Type for Entities

The first of these is our `BaseEntity` which we encourage you to inherit all of your entities from.  This type is located in the `Entities` folder, and defines a singular `Id` property, which in this case is a `Guid`.

``` csharp
public abstract class BaseEntity : IIdentifiable<Guid>
{
    public Guid Id { get; set; }
}
```

### What is IIdentifiable&lt;T&gt;?

Our sister project Highway.Data has a little known feature called PreBuilt Queries.  It defines a set of queries for you, including `GetById` and `FindAll`.  These queries use an interface `IIdentifiable<T>` to know what your Primary Key is.  **There is no requirement that your entities implement this interface, but if they do there are several features which are added.**  Note that the `T` defined the type of your key, and it can be changed to an `int` or any other type you prefer for your primary keys.

## Base Type for Mappings

We also provide a base type for all of your `EntityTypeConfiguration<T>` classes called `BaseMapping<T>`.  Aside from being a short name, which in and of itself is awesome, this class also maps your key and sets it to be database generated as an identity.  One important note is that the `T` for `BaseMapping<T>` is restricted being a `BaseEntity` so we can map the `Id` property.


```
public abstract class BaseMapping<T> : EntityTypeConfiguration<T> where T : BaseEntity
{
    public BaseMapping()
    {
        this.HasKey(e => e.Id);
        this.Property(e => e.Id).HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity);
    }
}
```

## Automatic Mappings Inclusion

Building on an awesome feature of Entity Framework 6.0, we automatically detect all `EntityTypeConfiguration<T>` classes in the MVC project.  This means you don't need to update our `IMappingConfiguration` every time you add a new entity.

```
public class HighwayMappingConfiguration : IMappingConfiguration
{
    public void ConfigureModelBuilder(DbModelBuilder modelBuilder)
    {
        modelBuilder.Configurations.AddFromAssembly(this.GetType().Assembly);
    }
}
```