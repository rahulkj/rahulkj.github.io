---
layout: post
title:  "Self Signed Certificates - Not standards compliant"
date:   2022-09-09 9:34:00 +0530
categories: openssl, certificates
---

If you are a Mac user, you must have noticed that you get an error with the cli (command line interface) tools written in golang, throw an exception:

`error: x509: "somehost" certificate is not standards compliant`

Now what is the problem here?

Back in the days, when someone used to generate self signed certificates using openssl, the number of days for which the certificate was requested for was set to larger number, like 3650 days (10 years).

We all know certificate management is a pain, and hence we tend to delay that pain by requesting a certificate that expires after certain years.

Apple recently changed the model that if a self signed certificate is greater than `825` days, then it qualifies the certificate as non compliant.

So if you in the same boat, then please regenerate the self signed ca and server certificates to limit them to expire in `825` days, and this should make the cli's happy.

If you need a script to generate the certificate then here is one:

https://gist.github.com/rahulkj/fa29996c7bdaa8c58a9edca2b3411b62

Hope this helped you!!

Cheers
