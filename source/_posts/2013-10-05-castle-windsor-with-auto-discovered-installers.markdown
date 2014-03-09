---
layout: post
title: "Castle Windsor with Auto Discovered Installers"
date: 2013-10-05 15:07
comments: true
categories: onramp-mvc feature
order: 10
---

# Configuring Castle Windsor

Dependency injection is a key part of good software development, it is the D or [SOLID], and yet is not something [Microsoft] ships a solution for.  As such, we have turned to the community and the long standing, well supported, king of IoC : [Castle.Windsor]

We configure Windsor with a set of reasonable defaults, and enable a few features which some people may not be aware of, to make it is painless as possible.  In our `WindsorActivator.Startup()` method, we do the following:

``` csharp
            // Create the container
            IoC.Container = new WindsorContainer();

            // Add the Array Resolver, so we can take dependencies on T[]
            // while only registering T.
            IoC.Container.Kernel.Resolver.AddSubResolver(new ArrayResolver(IoC.Container.Kernel));
            IoC.Container.AddFacility<TypedFactoryFacility>();

            // Register the kernel and container, in case an installer needs it.
            IoC.Container.Register(
                Component.For<IKernel>().Instance(IoC.Container.Kernel),
                Component.For<IWindsorContainer>().Instance(IoC.Container)
                );
```

We make your instance of [Castle.Windsor] available via `IoC.Container` but we have specifically marked that as `[Obsolete]` because you should not be accessing the container directly.  Instead, we want to encourage you to rely on our injection of dependencies into your Controllers, and never directly access the container outside of `App_Architecture`.

## Accessing the Container without an Obsolete Warning

Now, on rare occasions there will be perfectly reasonable pragmatic reasons to need to access the container.  This is the reason why we have marked the `[Obsolete]` merely as a warning, and then included the `#pragma` statements to disable those warnings in places you are accepting the need to directly access the container.  When those occur, simply surround your code with :

```
	#pragma warning disable 618
	#pragma warning restore 618
```

You should keep such sections small, and should be aware that the `#pragma` statements will also disable any other `[Obsolete]` warnings that occur between them, not just those for the `IoC.Container`.

# Discovering Installers

`WindsorActivator` also includes one very small, but very powerful line that is worth highlighting:

```
Container.Install(FromAssembly.This());
```

This statement tells [Castle.Windsor] to scan the **current assembly**, looking for classes which implement the `IWindsorInstaller` interface, and to execute those components, allowing them to register components with the container.  This auto discovery is one of the best features of our IoC implementation, allowing you to segregate your registrations into small, related pieces, as you will see.

# Included Installers

We have included several installers for you, in the various packages of Highway.OnRamp.MVC.   They are all located in the `Installers` folder of your MVC solution.  Here are their names and purposes:

* `CommonLoggingInstaller` - Configures the Common.Logging installer used by Highway.Data to log to NLog
* `ControllersInstaller` - Scans the current assembly for all types that implement `IController` from System.Web.Mvc and registers them with [Castle.Windsor].  This means you never have to register your controllers manually.
* `DefaultConventionInstaller` - Automatically registers types based on naming conventions, the [details of which we discuss in this feature](/blog/2013/11/06/automatic-ioc-registration-convention/).
* `EntityFrameworkInstaller` - Registers the configured DatabaseInitializer for Entity Framework.
* `HighwayDataInstaller` - Is the configuration and registrations for using Highway.Data.EntityFramework for your database.  It will be covered in more detail when we discuss Data Access.
* `LoggingInstaller` - Configures the Castle Logging Facility, and wires it to NLog.  We will cover this is more detail when we discuss the Logging feature.

[SOLID]:						http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[Microsoft]:				http://microsoft.com
[Castle.Windsor]:		http://docs.castleproject.org/Default.aspx?Page=MainPage&NS=Windsor&AspxAutoDetectCookieSupport=1
