---
layout: post
title:  "Maven Java Service Wrappers"
date:   2013-04-03 16:54:00 -0600
categories: ubuntu ldap
---

Maven Java Service Wrappers are useful for generating daemon scripts for the stand alone java programs.

By executing the generated scripts, the stand alone java program can be registered as a service in windows.

The generated scripts also provides various options like:
- `console`
- `start`
- `stop`
- `pause`
- `resume`
- `restart`
- `remove`
- `status`
- `install`

As the first step to use these generated scripts, we need to perform an install, after which we can start the application.

As a general tip for a stand alone app, ensure you provide sufficient memory to your application.

This is a fun plugin!!

### References:

- http://mojo.codehaus.org/appassembler/appassembler-maven-plugin/usage.html
