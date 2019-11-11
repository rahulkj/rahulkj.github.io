---
layout: post
title:  "Calculating IPs Available Using Bosh"
date:   2019-10-16 23:35:00 -0600
categories: bosh ips cloudfoundry
---

For big cloudfoundry customers, IP address management is a big challenge. It would be nice if there was a simple way to identify what IP's are available in a given CIDR block, so one could plan the network and determine bottlenecks due to IP Address allocations.

The logic to find the available ips, seems pretty straight forward:

```
total ips in the CIDR block - reserved IPs = available ips
```

But there's more to this. Determine if BOSH director belong to that CIDR, and is in the reserved range. If not, then take out one more IP.

Finally subtract the compilation worker vms if they belong to the same CIDR block.

Easy enough right :)

To gather this data, one needs to get the cloud config from bosh:

```
bosh cloud-config
```

Next, grab the ip's that have been allocated to the various deployments. Easy way to get this is by running

```
bosh vms
```

Once you have both the contents, the math becomes simple.

So overcome this complex process and doing the math, I ended up writing a utility in go. The link to the repo is [https://github.com/rahulkj/bosh-ip-util](https://github.com/rahulkj/bosh-ip-util)

Follow the instructions in the **README**, and rest is straight forward.
