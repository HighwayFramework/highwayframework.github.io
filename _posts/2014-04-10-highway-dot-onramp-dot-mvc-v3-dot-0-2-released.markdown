---
layout: post
title: "Highway.OnRamp.MVC v3.0.2 Released"
date: 2014-04-10 11:18
comments: true
categories: [onramp-mvc, release, hp]
---

Today we released a minor bug fix to the Highway.OnRamp.MVC packages, this included the following changes:

* Moving `BaseRestApiController` from Highway.OnRamp.MVC to Highway.OnRamp.MVC.Data.  This was an error in packaging previously which resulted in errors if you only installed Highway.OnRamp.MVC.
* Increased the dependency version of Highway.OnRamp.MVC.Data for Highway.Data from v5.0 to v5.0.6.  This removes the requirement for you to be using .NET Framework v4.5.1 and now merely required .NET Framework v4.5.

All other functionality is unchanged, all current documentation is still accurate.
