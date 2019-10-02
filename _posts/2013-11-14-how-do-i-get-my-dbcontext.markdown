---
layout: post
title: "How do I get my DbContext?"
date: 2013-11-14 11:10
comments: true
no_homepage: true
categories: [data, onramp-mvc, faq]
---

From time to time it is necessary to either pass a `DbContext` to some framework code that expects it, or to know the type of your `DbContext`, for instance when setting a database initializer.  The answer depends slightly on which part of Highway your using, as follows:

* In Highway.Data.EntityFramework, the `DataContext` class is a child of `DbContext`.  As such anywhere you need `DbContext`, simply pass your instance of `IDataContext` as `DbContext`.  This may require a soft cast if your variable is of type `IDataContext` and not `DataContext`, which it normally should and will be.
* In Highway.OnRamp.MVC.Data we subclass the default `DataContext` class of Highway.Data.EntityFramework into a class called `HighwayDataContext` to change the constructor a bit.  As such, the type of of your `DbContext` is `HighwayDataContext`, but as above any `IDataContext` can be cast to `DbContext` with success.
