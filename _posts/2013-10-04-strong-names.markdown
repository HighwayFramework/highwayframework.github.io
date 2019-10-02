---
layout: post
title: "Strong Names"
date: 2013-10-04 20:05
comments: true
categories: [data, pavement, hp]
---
We are thrilled to announce that we have today released an update to Highway.Pavement and Highway.Data which add strong name keys (SNKs) to the assemblies.  Furthermore, in keeping with good open source practices, the cryptographic keys to accomplish this signing are now checked into the source repository for each project.
<!--more-->
Thank you to friend of the project Michael Dudley for prompting us to do something which we should have done some time ago.

Please note, that since we've introduced these keys, there is a chance some Binding Redirects in your config files may need to be updated.  Here is the public key information if you should need it:

```
Microsoft (R) .NET Framework Strong Name Utility  Version 4.0.30319.17929
Copyright (c) Microsoft Corporation.  All rights reserved.

Public key (hash algorithm: sha1):
00240000048000009400000006020000002400005253413100040000010001000d9d9349bb0d52
9d9e45bda5d7c7e82852dfd8f8978b6499f712866db11623fb9ff9d616bb5e796f441b3ee5e681
16ba7f8a533a74e87d892226cb3dcf091b9fe476d41e862f5a5d8f768d5ced1c5816282ebdc7ef
8fb3d40709fb4e64b57141c1691f00dcc451a367671de255df03115b001923ec9590870772118e
2cbe65df

Public key token is fe803c9600455796
```
