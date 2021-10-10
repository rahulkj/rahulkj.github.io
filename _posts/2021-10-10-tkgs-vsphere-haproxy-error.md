---
layout: post
title:  "Error with TKG cluster provisioning"
date:   2021-10-10 16:51:00 -0600
categories: tkg, vsphere, k8s, homelab
---

## Create cluster failed

While provisioning a tkg cluster in vSphere, I ran into this error:

```
Error from server (unable to find a compatible full version matching version hint "1.21"): error when creating "cluster.yaml": admission webhook "version.mutating.tanzukubernetescluster.run.tanzu.vmware.com" denied the request: unable to find a compatible full version matching version hint "1.21"
```

After a lot of debugging, I realized some of the content library templates were not completed cloned, due to which the deployment of k8s clusters was failing.

I manually selected the templates that had no ova's synchronized from the remote server, and then waited for the process was complete. Repeated the same step for all the other ova's.

After all the templates were cloned down, I ran the command to create the cluster again, and there were no errors related to the above problem.

Hope this helped you!!

Cheers!