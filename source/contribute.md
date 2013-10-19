---
layout:  page
title: Contribute
---

# Our Mission Statement
## Our mission is to be the most **useful**, **approachable**, **testable**, and **well documented** framework for .NET developers.

We realize all to well that our mission is a work in progress.  Much of the documentation that is needed is not yet written, and the core team has taken a personal challenge to finish that work before the end of the year (2013).

### Useful

First and foremost, anything we add to the Highway Framework should be useful, addressing a clear need for a significant portion of developers.

### Approachable

Our components should be approachable, not requiring a PhD in Computer Science to understand or implement.

### Testable

Our components will always be developed in such a way as to make testing them as easy as possible.  In most cases, this will be through providing interfaces, but in some cases this will be accomplished by providing specific unit testing shims.

### Well Documented

Our components will be documented, feature by feature, to explain how to use what has been developed.

# Developing for Highway

## Developer Google Group

We maintain a Google Group for developers of the Highway Framework.  If you're interested, [please request membership to that group here.](https://groups.google.com/forum/#!forum/hwyfwk)

## Pull Requests

We love our contributors, and we love GitHub.  If you'd like to help contribute to the Highway Framework, then we encourage you to fork one of our repositories, and with the above in mind, submit a pull request.

## CI Server

We maintain a CI Server for Highway, in no small part thanks to the support of Microsoft and Jet Brains, that is always monitoring and rebuilding the packages with the latest changes.  You can login to that server at : http://hwyfwk.cloudapp.net/

## CI NuGet Feed

If you'd like to pull down our latest successful build, you can do so at : http://hwyfwk.cloudapp.net/guestAuth/app/nuget/v1/FeedService.svc/

Please remember that this is a Per Commit build feed, and it is entirely likely that certain new features will not be completely developed, or that errors could exist.  We only publish to the feed if all our tests pass, so if you discover something broken, feel free to send us a Pull Request with a broken test that highlights the problem.

The CI Feed purges down to only the last two successful builds every day, so if you're using it, do not expect that the version you pulled today will be there tomorrow.  Stability is what http://nuget.org releases are for. 

