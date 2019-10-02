---
layout: post
title: "Highway.Data v5.1.0 Released"
date: 2014-04-28 22:37
comments: true
categories: [data, hp, release]
---

* `DomainRepositoryFactory` doesn’t throw on null event collection – Eric Burcham
* Added `InMemoryActiveDataContext` to allow for multiple repositories using a single in memory – The beginnings of a “prod” in memory context – Long Mai
* Queued Add and Remove – `InMemoryDataContext` now acts more like a real database as it doesn’t return added records or modify collections in loops. – Long Mai
* Identity Strategies are invoked automatically on commit – Long Mai
* Brought back .NET 4.0 support
* Updated to Entity Framework 6.0.2
* Updated to CommonServiceLocator 1.2
* Updated to Common.Logging 2.2.0
