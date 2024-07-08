---
layout: post
title: "Upgrading TKGs on vCenter 8u3"
date: 2024-07-7 17:10:00 +0530
categories: kubernetes, vCenter, TKGs
---

Amazing new features come with vCenter 8 U3. Checkout the [release notes here](https://docs.vmware.com/en/VMware-vSphere/8.0/rn/vsphere-vcenter-server-803-release-notes/index.html)

### Upgrading workload management supervisor clusters
Once you have upgraded you vCenter to the latest version, and assuming you have the workload clusters feature turned on, you can follow the steps here to upgrade your supervisor clusters.

* Navigate to Workload Management >> Updates. Here you will notice your cluster listed, with the version of k8s it was provisioned with and then the latest available version
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-1.png)

* Select your cluster, and click on Apply Updates
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-2.png)

* This is when the pre-checks are performed, to ensure this cluster is compatible with the latest release and can be upgraded
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-3.png)

* Once the pre-checks complete, you can click on Next button
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-4.png)

* And review the upgrade message, and ensure you are ready to upgrade. Once ready, click on Finish.
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-5.png)

* At this point, the vCenter upgrades the k8s versions for all the supervisor nodes that are running on the hosts that are under the chosen cluster. This is time consuming activity
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-6.png)

* You can review the upgrade progress by clicking view hyperlink in the `Config Status` column
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-7.png)

* Once the upgrade has completed, you will notice the column in `Config Status` will show as Running. If something has gone wrong, then you will need to troubleshoot the error. Most likely a restart of you host can help resolve the issue.
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-8.png)

* Once upgrade is complete, you will notice additional 1 namespaces get provisioned, which is for velero backups.
![]({{ site.url }}/assets/tkg-v8u3-upgrade/tkg-upgrade-9.png)

My upgrade was painless, but I will update this blog when I run into any other issues. Happy learning!!