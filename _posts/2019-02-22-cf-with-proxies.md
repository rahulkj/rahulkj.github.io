---
layout: post
title:  "Proxies with cloud foundry"
date:   2019-02-22 18:40:00 -0600
categories: cloudfoundry proxies
---

One of the most annoying topics in my work experience, is dealing with proxies. I understand why customers use proxies, but when it comes to a product like cloud foundry, where there are multiple components that need to talk, so they can function successfully, then with proxy setup, it becomes hard.

As a thumb-rule, I suggest setting the 3 variables:

```
HTTP_PROXY=http://company.com:80
HTTPS_PROXY=http://company.com:80
NO_PROXY=192.168.10.0/26,192.168.12.0/23,192.168.14.0/23,.pcf.compamy.com
```

where,
- `192.168.10.0/26` - is the management network
- `192.168.12.0/23` - is the deployment network
- `192.168.14.0/23` - is the services network
- `.pcf.company.com` - is the top level domain used for the system and apps

Ensure you list all the internal subnets in the `NO_PROXY`, so you don't have to sweat on debugging issues related to proxies

Hopefully, this helps all the Platform Engineers who are operating **Pivotal Cloud Foundry**!
