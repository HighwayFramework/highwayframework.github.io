---
layout: post
title: "Does Highway.Data work with Database First?"
date: 2013-11-07 15:32
comments: true
no_homepage: true
categories: data faq
---

## Short Answer 
Yes, sort of.

## Long Answer
Inject-able mappings and context configuration are not offered, for obvious reasons. Otherwise in this version all other features are supported.

## A word about the future

In future versions of Highway.Data, it is highly likely that features will be introduced that are not compatible with Database First.  How we handle this will be decided later, but this is not a feature which we are committed to fully supporting, it is provided as a **basic** scenario.

### Entity Framework Power Tools

As a quick word, most people who have chosen Database First, did so because they believed that Code First would require that their classes create, or otherwise control their database.  This is not the case.  [The Entity Framework team has for some time shipped a set of Power Tools](http://visualstudiogallery.msdn.microsoft.com/72a60b14-1581-4b9b-89f2-846072eff19d) which allow the reverse engineering of existing database into Code First entities and mappings, which are then fully compatible with Highway.Data.  We encourage those currently using Database First to migrate using these tools to a Code First solution.



