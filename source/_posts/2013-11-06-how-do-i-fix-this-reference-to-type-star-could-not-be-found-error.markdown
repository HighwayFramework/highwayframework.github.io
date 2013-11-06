---
layout: post
title: "How do I fix this 'Reference to type * could not be found' error?"
date: 2013-11-06 07:49
comments: true
no_homepage: true
categories: data faq
---
In Highway.Data 4.1  we added strong name keys (SNKs) to all assemblies.  This was a good thing, but bad version number handling and unfortunately can't be fixed now that it's in the wild.  This most often presents itself as an error similar to :

```
Reference to type 'Highway.Data.IContextConfiguration`1' claims it is defined in 'C:\source\MyProject\packages\Highway.Data.4.0.5.3\lib\net40\Highway.Data.dll', but it could not be found
```

The particular type that it complains about (`IContextConfiguration<T>` in this case) may vary case to case, but the combination of a `Reference to type * could not be found` and that error references a version prior to v4.1 are the signs that you've encountered this bug.

## Resolution

Unfortunately, any components that depended on the pre-v4.1 assemblies will need to be recompiled.  Because the SNK was added there is no Assembly Binding Redirect or other solution to this, we suck, and we're sorry.  Rest assured that all future Highway Framework assemblies will be signed from day one and that we've learned this lesson.
