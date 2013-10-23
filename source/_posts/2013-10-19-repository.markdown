---
layout: post
title: "Repository"
date: 2013-10-19 08:17
comments: true
order: 20
categories: data feature
---

{% pullquote %}
The [Repository Pattern](/blog/2013/10/19/understanding-the-patterns/) describes an intermediary between code which uses data, and the particulars of how that data is retrieved.  In Highway.Data, we have a `Repository` class which is meant for this exact purpose, and it implements an `IRepository` interface which your code should take as a dependency.  {"IRepository is the singular interface which your business code should be aware of from Highway.Data."}  Through it we can accomplish any database operation that is needed.
{% endpullquote %}

In Highway.Data, we define our `IRepository` interface as follows:

``` csharp
    public interface IRepository
    {
        IDataContext Context { get; }
        IEventManager EventManager { get; }
        
        IEnumerable<T> Find<T>(IQuery<T> query);
        T Find<T>(IScalar<T> query);
        void Execute(ICommand command);
    }
```
 
The interface first has two properties, those being `Context` and `EventManager`.  The details of those are covered elsewhere, though we will touch lightly on `Context` in this article.  The other three methods are very interesting, and each deserve a section of their own.

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


# Find Multiple Things - IEnumerable&lt;T&gt; Find&lt;T&gt;(IQuery&lt;T&gt; query)

This method is used when you want to retrieve multiple things from the database.  Let's assume we had a MVC controller which wanted to query for all drivers who had taken a class from Instructor #5.  First thing we'll need is a controller which has a dependency of our `IRepository` interface.

```
public class DriversController : Controller
{
    private IRepository repo;

    public DriversController(IRepository repo)
    {
        this.repo = repo;
    }
}
```

Now then, we'll add an action method to this controller, which is called `ByInstructor` which takes an `int` of the Instructor's `Id`.  Then we call to the repository, passing in an instance of our Query object called `FindDriversByInstructorId`.  Like so:

```
public class DriversController : Controller
{
    private IRepository repo;

    public DriversController(IRepository repo)
    {
        this.repo = repo;
    }

    public ActionResult ByInstructor(int instructorId)
    {
        IEnumerable<Driver> drivers = repo.Find(new FindDriversByInstructorId(instructorId));
        return View(drivers);
    }
}
```

Note how our `DriversController` is not aware of anything database specific, like connection strings, or even which Object Relational Mapper (ORM) is servicing the request.  All it knows is that it is asking for multiple drivers, and receiving them.  Now, you might be curious what's going on inside of `FindDriversByInstructorId`, and while we're not discussing queries here, I want to show you the code so you don't think there is magic going on:

```
public class FindDriversByInstructorId : Query<Driver>
{
    public FindDriversByInstructorId(int instructorId)
    {
        ContextQuery = c => c.AsQueryable<Instructor>().SelectMany(e => e.Drivers);
    }
}
```

# Find One Thing - T Find&lt;T&gt;(IScalar&lt;T&gt; query)

Our next method is used when we want to retrieve a singular thing from the database.  That one thing might be an `int`, for instance a count of rows, or it might be an entity like `Car`, for instance getting the car for a particular driver.  This time we'll be working with the `CarController`, and we're going to retrieve a `Car` by it's `Driver`'s `Id`.  Our controller would use `IRepository` like so:

```
public class CarController : Controller
{
    private IRepository repo;

    public CarController(IRepository repo)
    {
        this.repo = repo;
    }
    
    public ActionResult ByDriver(int driverId)
    {
        Car car = repo.Find(new FindCarByDriverId(driverId));
        return View(car);
    }
}
```

As you can see, we return a singular object, once again completely unaware of which ORM is doing the work, or anything related to the database.  Like before, while we're not discussing Scalars here, this is the query being used :

```
public class FindCarByDriverId : Scalar<Car>
{
    public FindCarByDriverId(int driverId)
    {
        ContextQuery = c => c.AsQueryable<Driver>()
            .Where(e => e.Id == driverId)
            .Select(e => e.Car)
            .FirstOrDefault();
    }
}
```

# Do Something - void Execute(ICommand command)

Our final method is used to perform a command that has no return value.  There are often times when you want the database to do some work, but you don't need it to return anything as a result of that work, this is what `Execute` is for.  In our example, we're going to add another method to our `CarController` class from the previous example, which will remove all Chevy vehicles from our database.  Our `CarController` will now look like this:

```
public class CarController : Controller
{
    private IRepository repo;

    public CarController(IRepository repo)
    {
        this.repo = repo;
    }
    
    public ActionResult ByDriver(int driverId)
    {
        Car car = repo.Find(new FindCarByDriverId(driverId));
        return View(car);
    }

    public ActionResult RemoveChevy()
    {
        repo.Execute(new DropMake("Chevy"));
        return RedirectToAction("Index", "Home");
    }
}
```

Note how `Execute` returns no value.  Our `DropMake` command looks like this :

```
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

As the comment says, this is an inefficient way to remove multiple rows, please review the documentation on `AdvancedCommand` if you really need to perform a delete like this.