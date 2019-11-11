---
layout: post
title:  "CloudFoundry Monitoring Dashboard"
date:   2016-03-09 10:41:00 -0600
categories: cloudfoundry monitoring
---

If you're looking for an application that can pull the details from `Ops Metrics` and `Cloud Controller`, and display it on a webpage, then you would want to checkout [foundation-metrics](https://github.com/pivotalservices/foundation-metrics)

This application has 2 components, one backend and one frontend. The backend component can send emails, if configured during the `cf push`

The frontend application just consumes the data from the rest services, and displays them. The UI refreshes data every `60s`
