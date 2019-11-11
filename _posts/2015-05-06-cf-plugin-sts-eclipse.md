---
layout: post
title:  "Cloudfoundry plugin with STS/Eclipse with Self Signed Certificates"
date:   2015-05-06 13:40:00 -0600
categories: cloudfoundry development sts eclipse
---

I'll keep this short.

If you use STS/eclipse, and you're looking to connect to your cloudfoundry instance, that uses a self signed certificate, then follow the steps below to get around the SSLHandShake errors:

- copy the public certificate by running `openssl s_client -showcerts -connect api.<system-domain>:443`

- `sudo keytool -import -alias alias -keystore <JAVA_HOME>/jre/lib/security/cacerts -file  <PATH to the certificate>`

- Restart STS, and now try using the cloudfoundry STS plugin

If you've followed the steps, you should now be able to connect to your cloudfoundry instance with your username/password.
