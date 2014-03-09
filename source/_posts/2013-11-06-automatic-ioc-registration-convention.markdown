---
layout: post
title: "Automatic IoC Registration Convention"
date: 2013-11-06 09:27
comments: true
order: 11
categories: onramp-mvc feature
---

{% pullquote %}
In order to ease the pains of registering types to an IoC container, we include a `DefaultConventionInstaller` which registers components automatically to the IoC container based on naming conventions. {"The class Foo will be automatically registered to interface IFoo"}, provided of course that it implements IFoo.  This is accomplished using Windsor's registration conventions [which are documented here](http://docs.castleproject.org/Windsor.Registering-components-by-conventions.ashx).  By default we register all components with a Lifestyle of `PerWebRequest`, so one instance will be created per web request.  If you need to register a component with a different lifestyle than this, you will also need to exclude it from the default convention.  You can see an example of this below, where we exclude anything that is an `IController` because they are registered with a Transient Lifestyle in another installer
{% endpullquote %}

``` csharp
public void Install(IWindsorContainer container, IConfigurationStore store)
{
    container.Register(
        Classes.FromThisAssembly().Pick()
            .WithServiceDefaultInterfaces()
            .Unless(type => typeof(IController).IsAssignableFrom(type))
            .LifestylePerWebRequest()
        );
}
```