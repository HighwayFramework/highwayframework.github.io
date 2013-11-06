---
layout: post
title: "Naming Conventions"
date: 2013-11-06 11:37
comments: true
no_homepage: true
categories: onramp-mvc feature
---

Starting in version 3.0 of Highway.OnRamp.MVC we have made a sincere effort to clarify naming conventions so you can understand the purpose of each piece of the OnRamp.  We have several important naming and organization conventions.

## Naming Conventions

### Activators

Activators are classes that run when a process is starting up.  For Highway.OnRamp.MVC this means as part of the `HttpApplication` cycle.  [We talk about WebActivator and how we accomplish this elsewhere](/blog/2013/10/05/webactivator-for-composable-startup/), but if a class name ends with `Activator` you know it runs during application startup.

### Configs

Interfaces where the name ends with `Config` are configured automatically at run-time to provide values from `web.config`.  This provides you the ability to mock these quickly in tests, and not litter your code with "magic strings" for accessing configuration values.

### Sessions

Interfaces where the name ends with `Session` are configured automatically at run-time to provide values from the current ASP.NET Session.  This provides you the ability to mock these quickly in tests, and not litter your code with "magic strings" for accessing session variables.

### Installers

Classes that implement `IWindsorInstaller` are named with the `Installer` suffix and are used to register components with the IoC.  Please note there is no guarantee of order of execution of these installers, but they do run at the end of the `WindsorActivator` so anything it sets up can be assumed.

## Organization Conventions

### App_Architecture

The `App_Architecture` folder contains all our structure related to startup events, IoC registration, concrete classes for registered interfaces, etc.  We believe that we've achieved an organization structure that your business code should never need to take a `using` statement to things within this folder.  This is the self-contained area which handles all of the wire-up aspects of the OnRamp, and while it will effect how your app runs, the classes should not need to be directly referenced.

#### App_Architecture \ Services
This folder contains a `Core` folder for classes support Highway.OnRamp.MVC, but then also contains folders for each plug-in to the OnRamp which contains their classes.  For instance, if you bring in Highway.OnRamp.MVC.Data it will also contain a `Data` folder which has classes required by that plug-in.

### Entities & Mappings

If you bring in Highway.OnRamp.MVC.Data you will get an `Entities` folder, and it's sub-folder `Mappings`.  This is meant to contain all of your Entity Framework entities, as the `Mappings` folder is meant to contain classes which implement `EntityTypeConfiguration<T>` classes that configure Entity Framework on how to map those entities.


