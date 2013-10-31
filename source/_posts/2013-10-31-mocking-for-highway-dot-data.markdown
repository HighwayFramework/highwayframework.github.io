---
layout: post
title: "Mocking For Highway.Data"
date: 2013-10-31 06:32
comments: true
no_homepage: true
order: 22
categories: data feature
---

After struggling for quite some time with testing different abstracts for Data Access, we came to realize that they all had widely varying support and inconsistent levels of mocking. This caused trouble testing, and could driver way to much data access specific knowledge into my logical tests. To remedy this we included 2 standard layers of mocking for testing classes that leverage Highway.Data.

#Example Domain

For the below examples we will be working with the following controller and testing it properly.

``` csharp
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

#Repository Mocking

Consider a test for the controller above that needs to test the implementation of the ByDriver or RemoveChevy calls without testing the data access as well. We can accomplish this with the following test.

``` csharp

```

Notice how we can just mock the returns of the `Query<T>`. This gives us the ability to test that the logic is correct in asking for the right data, but not mix in the concern about the data access being correct in it's implementation. We will cover how to test and mock those slightly ]lower in this post](mockingContext).


<a name="mockingContext"></a>
#Context Mocking