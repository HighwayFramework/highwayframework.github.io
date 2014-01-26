---
layout: post 
title: "Session and Configuration access via Interfaces"
date: 2014-01-25 19:33
comments: true
order: 30
categories: onramp-mvc feature
---


There are two things which are historically very hard to deal with when unit testing a websites, these are configuration values, and session variables.  They both present largely the same problem, they are normally accessible via an API that is not conducive to testing, both are static classes, and that API retrieves data from them via "magic strings".  The Castle Project has a wonderful component within it called `DictionaryAdapter` which can help solve these pains for us very easily, once you understand how it works.

# Dictionary Adapter

Dictionary Adapter is a class which creates classes at runtime that implement an interface you specify, and wires the properties of said interface to access a key value store you specify.  For instance, let's assume you have the following interface:

```
public interface IConnectionStringConfig
{
	string ConnectionString { get; }
}
```

And the following configuration settings in your web.config:

```
  <appSettings>
    <add key="ConnectionString" value="Server=.;Database=ChangeMyConnectionString;Integrated Security=true;" />
  </appSettings>
```


Now we can ask Dictionary Adapter to create an instance of `IConnectionString`, and supply it the app settings from our web.config.  It will return to us a class which implements our interface, and when that when the `ConnectionString` property is accessed, will retrieve the value from app settings.


Now instead of writing controllers, or other classes which access app settings directly from the `ConfigurationManager`, we can instead simply take a dependency on `IConnectionString`, and that class can in turn use a strongly typed (no magic strings) and testable (simply mock the interface) way of accessing app settings.

Just as we can do this with the key value store of app settings, we can also do it with the key value store of `Session` as well.  This makes accessing session also no longer require magic strings, and addresses the problem of mocking an `HttpContext` for `Session`.

# Wiring it up with Windsor

As awesome as that is, manually registering all of the configuration or session interfaces would be rather tedious in the extreme.  As such we've created some conventions you can use that will automatically register these interfaces for you.

## Config interfaces

Any interface in your application's assembly which has a name that ends in **Config** will automatically be registered with our IoC container with a class instance provided by `DictionaryAdapter`.  These classes are registered as Singleton lifestyle, because they continue to directly access the `ConfigurationManager` and so even if your configuration values change at runtime, they will stay up to date.

## Session interfaces

Any interface in your application's assembly which has a name that ends in **Session** will automatically be registered with our IoC container with a class instance provided by `DictionaryAdapter`.  These classes are registered with a `PerWebRequest` lifestyle, because every request will have a different `HttpContext`.

# What about keys which don't match property names?

A common scenario for users of DictionaryAdapter to run into is that they want properties on their interfaces which do not match their key value store's key values exactly.  This is easy to run into simply because of the limitations of property names, for instance "ida:ClientSecret" is not a valid property name.  So how do we handle this?

DictionaryAdapter provides a set of attributes which can help address these concerns.  We'll talk about each in turn, here they are:

* `KeyAttribute`
* `KeyPrefixAttribute`
* `KeySuffixAttribute`

## Key

The simplest and often most useful of our helper attributes is `KeyAttribute`.  This attribute when applied to a property completely changes the key in the key value store that is accessed for our property.  For instance, given the following interface, the `ConnectionSting` property returns "Foo" setting instead of "ConnectionString" because of the `KeyAttribute`.

```
public interface IExampleConfig
{
	[Key("Foo")]
	string ConnectionString { get; }
}
```


## KeyPrefixAttribute

The next most common attribute to be used, and one which we use by default in the Data extension to the OnRamp, is `KeyPrefixAttirbute`.  This attribute is applied to the interface, not the properties, and affects the keys used in the key value store for all properties on that interface.  For instance, given the following interface, the ConnectionSting property returns "FooConnectionString" setting instead of "ConnectionString" because of the `KeyPrefixAttribute`.

```
public interface IExampleConfig
{
	[KeyPrefix("Foo")]
	string ConnectionString { get; }
}
```

## KeySuffixAttribute

The third and final attribute we will discuss here is `KeySuffixAttribute`.  Like `KeyPrefixAttribute` this is applied to the interface, not the property, and affects the key values for all properties on that interface.  For instance, given the following interface, the `ConnectionSting` property returns "ConnectionStringFoo" setting instead of "ConnectionString" because of the `KeySuffixAttribute`.

```
public interface IExampleConfig
{
	[Key("Foo")]
	string ConnectionString { get; }
}
```
