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
 
#Adding Items
When you add an item there are several things that could/should happen. Knowing at which point the failure happens is important. This is where Highway Framework makes things easier on you. 