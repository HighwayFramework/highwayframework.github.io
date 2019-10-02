---
layout: post
title: "Highway.Data v5.0 Released"
date: 2014-01-25 16:40
comments: true
no_homepage: false
categories: [data, release, hp]
---
#Highway.Data 5.0 Released ( 5.0.1.0 )
##New Features, and better Domain Driven Design support

###Domain Context

We have added support for advanced domain specific contexts. These allow for detailed manipulation of your data access patterns. It adheres to the Domain Driven version of a bounded context.

####More Details are here: [Domain Context](/blog/2014/01/25/domain-context/)

###Simplified Creation Pattern

We have also added the much requested `Repository` factories. These give you the ability to use Highway.Data in services and long living work-flows without having to hand roll your own management of unit of work.

####More Details are here: [Creation Patterns](/blog/2014/01/25/simplified-creation-patterns/)

###Entity Framework 6.0.0.2 Support

We shipped an EF 6.0 supported version now that some of the initial release bugs have been patch. We did require the 6.0.0.2 version so that we could make sure you get their fixes as well.

We will be working over the next 3-4 weeks to support some of the new features ( that we don't already support) in Entity Framework 6.0.

####More Details are here:  [EF 6.0](http://msdn.microsoft.com/en-us/data/ee712907#ef6)

##Breaking Changes:

###AsQueryable off Repository.Context

After seeing a lot of examples from clients, friends, and other speakers using `repository.Context.AsQueryable()`, we realized that this represented a large hole in the design pattern for Query Object separation. We have remove that hole this publish. If you were using this for simple operations please look at the pre-built queries, or codify the queries into `Query<T>, Scalar<T>, Command<T>` objects.

###Interception
We have disabled event interception for the simple `DataContext` as it caused an additional understanding and performance overhead. This is in an effort make sure we have a simple entry story for most developers. If you were using this feature, we have good news. We have expanded the feature with pre/post `Command`, `Scalar`, and `Query` execution intercept points, as well as kept the standard pre/post save. These features have moved onto the `DomainContext<T>` class due to their advanced nature.

####More Details are here:

##Thanks to:

####Curtis Schlak for pushing us to get EF 6.0 support published.

####Michael Dudley for the design sessions on vacation in Las Vegas, and the push to finish the beta release.
