---
layout: post
title:  "Concourse jobs are waiting too long before kicking off"
date:   2021-03-03 10:00:00 -0600
categories: concourse, ci
---

Have you encountered an issue where you triggered the build in concourse, and there is a lag in the time when the build starts?

If yes, then possibly your concourse environment needs a restart of the web and worker vms.

Recently, I encountered an issue where I committed a code in git, and the pipeline check resource pulled the new version right away. But to my surprise, the build took 10 minutes to trigger the pipeline. 

Initially, I ignored this and assumed it was just one off those days. But this continued to be the case with all the builds thereafter. 

To investigate this, I went through the issues list on concourse and found there were known issues with the release that I had, so had to be something unique in my environment.

Since I used `bosh` to create my concourse deployment, I went on to `ssh` onto the worker vms, and the web vms.

I tried checking the date on these vms, and to my surprise, the web vm `date` was not in sync with that on the worker vm/s. 

So I did a `bosh recreate` of the web vm, and guess what, that was it!!

I again went on to `ssh` onto the web vm to validate if the `date` is consistent with other vms, and now it was.

I did a test commit, and now the pipeline just took off instantaneously. Hope this helps you fix your latency issues with the pipelines.

Again, this was just one of several possiblities, but the above solution did the trick for me.

Put on your debugging hats and fix things!!!

Cheers!