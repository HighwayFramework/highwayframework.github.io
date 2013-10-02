---
layout: default
---

# [Project Home][Home] | [Getting Started][Start]

## Getting Started with Highway.OnRamp.MVC

### Create an MVC Site

Our first step will be to create an MVC site, or open an existing MVC project.

### Install the Package

Next open the **Package Manager Console** and issue the command:

```
Install-Package Highway.OnRamp.MVC.All
```

### Create Value, Not Boilerplate

That is all.  You should not be able to build and run your site, and a wide variety of features should be automatically engaged.  In the next section we'll introduce those features, one by one, and explain roughly how they work.

## Features Tour

### Web Activator

Throughout the rest of the features for this OnRamp, you will see we perform a lot of work just before, or just after, App_Start runs.  In order to be able to compose these behaviors into your project, we use a project called WebActivator.  That sounds very complex, but it's very simple, there are attributes which can be used to make a particular static method be called either just before or just after App_Start.

We only run one thing before App_Start : **Bootstrapping Castle.Windsor**.  Basically we create our container instance, and register all components.  This means that from App_Start onward you can count as given that an IoC container exists, and all provided registrations have been run.

### Dependency Injection

The OnRamp introduces the ability to do Dependency Injection via Castle.Windsor, a very popular and well understood project from the Castle Project.  In the class `IoC` there exists an instance of the `IWindsorContainer`.  **Please note it is marked as obsolete**!! You should not access this directly, unless you're writing code which fires at App_Start.  In all other cases, the fact that we handle injecting Controllers should ensure you never need to resolve anything directly.

#### Registering with Castle
As part of the bootstrapping process, all classes which implement `IWindsorInstaller` from the current application are executed.  The OnRamp creates several of these, and locates them in the `Installers` folder.


[Home]:				/projects/onramp/mvc/
[Start]:				/projects/onramp/mvc/start.html
[Data]:				/projects/data/
[Insurance]:		/projects/insurance/
[OnRamper]:			/projects/onramper/
[MVC]:				/projects/onramp/mvc/
[Services]:			/projects/onramp/services/
[Pavement]:			/projects/pavement/
[RoadCrew]:			/projects/roadcrew/
[Configuration]:	/projects/configuration/