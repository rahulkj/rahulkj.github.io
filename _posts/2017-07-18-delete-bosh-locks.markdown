---
layout: post
title:  "Deleting BOSH locks"
date:   2017-07-18 21:19:00 -0600
categories: bosh locks
---

If you encounter the error `"Error 100: Unable to get deployment lock, maybe a deployment is in progress. Try again later."`, then likely there is something going on with BOSH

- there is a deployment running
- bosh DB is locked, maybe due to a catastrophic failure of the IaaS

If bosh tasks, doesn't output any running tasks, then perform the following

Steps to delete locks from director:

```
> ssh -i ~/bosh vcap@<BOSH-DIRECTOR-IP>

> cd /var/vcap/packages/postgres/bin

> ./psql -U postgres -p 5432 bosh

> delete from locks;

> \q

> exit
```

Now you should be able to execute your bosh deployments without seeing the error
