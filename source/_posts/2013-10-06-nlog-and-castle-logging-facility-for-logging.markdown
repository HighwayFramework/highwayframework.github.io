---
layout: post
title: "NLog and Castle Logging Facility for Logging"
date: 2013-10-06 08:50
comments: true
categories: onramp-mvc feature
order: 20
---

One of the most important features of any application is a good logging strategy.  Because of this, `Highway.OnRamp.MVC.Logging` brings together two very well established projects to provide a world class logging experience.  First, we use the [Castle Logging Facility][clf] to provide a common abstraction over logging, so you are not coupled to a specific logging framework.  Then we implement [NLog] to perform the actual logging, configuring it via the `NLog.config` file.

# How do I log from a class?

If you want to log from a class, there is a very simply pattern we recommend you follow:

``` csharp
using Castle.Core.Logging;

public class MyNewClass
{
	public MyNewClass()
	{
		Logger = NullLogger.Instance;
	}
	
	public ILogger Logger { get; set; }
}
```

The steps are quite simple:

* Declare a public property of type `ILogger`
* Initialize that property in your constructor to `NullLogger.Instance`

This uses the ability of Castle.Windsor to do Property injection, so that after you class is constructed Castle Windsor will assign the real logger to that `Logger` property.  But, if you ever remove the `LoggingInstaller` your code will continue to work because the `NullLogger` is a proper implementation of `ILogger` which does nothing.  This avoids `NullReferenceException`s when you don't have a Logger injected, for instance during tests.

# How does that get setup to begin with?

The Logging Facility is setup in the `LoggingInstaller` class, where we configure it.  A facility is a Castle.Windsor concept for a bundled set of registrations and behaviors.  As you can see, this makes properly configuring something like NLog very easy:

``` csharp
public class LoggingInstaller : IWindsorInstaller
{
    public void Install(IWindsorContainer container, IConfigurationStore store)
    {
        
        container.AddFacility<LoggingFacility>(m => m.UseNLog().WithConfig("NLog.config"));
    }
}
```

That really is all there is to it, we configure the facility to use NLog, and then tell it where the config file is.  Now, the config file we include is very simple:

``` xml
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      globalThreshold="Debug">
  
  <targets>
    <target xsi:type="File"
            name="file"
            fileName="${basedir}/Logs/Log.txt"
            archiveEvery="Day" />
  </targets>

  <rules>
    <logger name="*" minlevel="Debug" writeTo="file" />
  </rules>
</nlog>
```

This configuration sets up a `Logs` directory that will contain `Log.txt` as a file which will be archived to another file every day.  It also configures NLog to log all messages Debug and above by default.  Not that while NLog supports a Trace level, below Debug, Castle Logging Facility does not so this essentially says "log everything to Logs.txt".  There is a great deal more that can be done with NLog and I encourage you to [review their documentation on config][nlogconfig].

# What is logged automatically for me?

We setup two types of loggers for you automatically;

* The App_Start component `LoggerAnnouncementsWireup` logs a message every time your application starts up, and if it properly shuts down.
* The filter `ExceptionLoggingFilter` logs every exception that is unhandled by the application.


[clf]:						http://docs.castleproject.org/Windsor.Logging-Facility.ashx
[NLog]:						http://nlog-project.org/
[nlogconfig]:			https://github.com/nlog/nlog/wiki