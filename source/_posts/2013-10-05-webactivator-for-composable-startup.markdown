---
layout: post
title: "WebActivator for Composable Startup"
date: 2013-10-05 15:09
comments: true
categories: onramp-mvc feature
---

The first and most important piece our OnRamp is the use of [WebActivator] to allow us to declare classes that will run either just before or just after the App_Start event of your ASP.NET application, or when your application shuts down.  This is accomplished using an assembly level attribute, pointing to a particular static class and method.

Consider the following class declaration from our `IoC.cs` file:

``` csharp
[assembly: WebActivator.PreApplicationStartMethod(typeof(Templates.App_Start.IoC), "Startup")]
namespace Templates.App_Start
{
    public static class IoC
    {
        public static void Startup()
        {
          // Details here described in the next feature…
        }
    }
}
```

Note how we declare the `PreApplicationStartMethod`, and point to the type of `Templates.App_Start.IoC` and then the `"Startup"` string lets it know which method to run.  This results in our `IoC.Startup()` method being called before App_Start.

Just as there is `PreApplicationStartMethod`, there is also `PostApplicationStartMethod` and `ApplicationShutdownMethod`, both of which are used in our Logging wire up which will be discussed later.


[WebActivator]:		http://www.nuget.org/packages/WebActivatorEx/