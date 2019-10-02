---
layout: post
title: "MVC Filters with Dependency Injection"
date: 2013-10-06 20:39
comments: true
categories: [onramp-mvc, feature]
order: 30
---

One of the most powerful features of the MVC OnRamp is the ability to have MVC Filters injected via our Inversion of Control (IoC) container, Castle.Windsor.  There are several types of filters supported by ASP.NET MVC that allow you to handle orthogonal issues such as:

* `IExceptionFilter` which is invoked whenever unhandled exceptions occur.
* `IActionFilter` which is invoked just before and just after specific actions.
* `IAuthorizationFilter` which is invoked when authorizing requests.
* `IResultFilter` which is invoked on just before and just after results are returned.

Normally these filters are applied one of two ways:

* Globally via specification in the App_Start, which can easily be resolved from the IoC container but which must be global in scope, and hence somewhat limiting.
* Via Attribute on either a Controller or Action, which cannot be resolved from the IoC because we have no control over the instantiation of those attributes.

But MVC allows for another option, which is an `IFilterProvider`, this interface is called at the outset of any request, and is allowed to return at runtime instances of filters which are to be applied.  Using this interface, we have created a class that resolves filters from within Castle.Windsor.  Consider the following class:

``` csharp
public class IoCFilterProvider : IFilterProvider
{
    private readonly IEnumerable<Func<ControllerContext, ActionDescriptor, Filter>> registeredFilters;

    public IoCFilterProvider(Func<ControllerContext, ActionDescriptor, Filter>[] registeredFilters)
    {
        this.registeredFilters = registeredFilters;
    }

    public IEnumerable<Filter> GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)
    {
        return registeredFilters.Select(m => m.Invoke(controllerContext, actionDescriptor)).Where(m => m != null);
    }
}
```

This simple class takes as a dependencies an array of delegates, speficically an array of `Func<>` delegates which receive as parameters the `ControllerContext` and `ActionDescriptor` and which return an instance of the `Filter` class.

* The `ControllerContext` class describes the controller that is about to be called.
* The `ActionDescriptor` class describes the action on that controller which is about to be called.

Given this information, you can decide to either return a `Filter` which will be applied, or return a null which will take no action.

# How do I register a filter?

In our `FilterInstaller` class you will see an example of registering such a filter:

``` csharp
public class FilterInstaller : IWindsorInstaller
{
    public void Install(IWindsorContainer container, IConfigurationStore store)
    {
        container.Register(
            Component.For<IFilterProvider>().ImplementedBy<IoCFilterProvider>(),
            Component.For<ExceptionLoggingFilter>().ImplementedBy<ExceptionLoggingFilter>(),
            Component.For<Func<ControllerContext,ActionDescriptor,Filter>>().Instance(
                (c,a) => new Filter(container.Resolve<ExceptionLoggingFilter>(), FilterScope.Last, int.MinValue))
            );
    }
}
```

On line 8, we register a `Func<ControllerContext,ActionDescriptor,Filter>` and state that we will provide a specific instance of that delegate to be used.

On line 9, we use the lambda syntax to declare a delegate, which is provided `(c,a)` as the parameters of type `ControllerContext` and `ActionDescriptor`, and which has a body of:

``` csharp
new Filter(container.Resolve<ExceptionLoggingFilter>(), FilterScope.Last, int.MinValue)
```

In this simple case, we create an instance of the `System.Web.Mvc.Filter` class and provide it our `ExceptionLoggingFilter` resolved from the container, and then tell the Filter to run in the `FilterScope.Last`, aka run this filter after all others, and order it within that scope using the `int.MinValue`, aka I really mean last of all last filters.

You can easily extend these registrations to include other filters by simply registering their delegates with the container and deciding when to return an instance of `Filter` and when to return `null` based on your business need.  Our example always returns, because we want to always log exceptions, but that is not required.  If your delegate examines the input data and determines it does not need to run a filter, simply return `null`.
