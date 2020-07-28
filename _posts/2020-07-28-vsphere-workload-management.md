---
layout: post
title:  "vSphere 7 + Workload Management"
date:   2020-07-28 5:00:00 -0600
categories: tkg, vsphere 7, workload management, homelab
---

To enable Workload Management on vSphere 7, ensure the following steps are completed:

* Create a VDS without any porrt groups, and attach the desired hosts to it, allocate the vmnic
* Setup NSX-T Management, Edge and prepare the TEP on the VDS
* Create a Tag on the storage/s that will be used for workload management
  - Click in vCenter > Storage
  - Select the storage and click on Actions > Tags & Custom Attributes > Assign Tag
  - Create a tag with the name `k8s` and Category name as `k8s` and select all
  - Assign the tag
* Next create a `VM Storage Policies` by clicking in vCenter > `Policies and Profiles` > `VM Storage Policies`
  - Click on `Create VM Storage Policy`
  - Name: `k8s-storage-policy`
  - Enable tag based placement rules
  - Tag Category: `k8s` and Tags `k8s`
  - Storage compatibility: should see all the storages
* On the clutser, enable DRS and HA. Ensure the vmk0 is enabled for vMotion & vSAN
* Deploy T0-Router

* If all the above is done, then click on vCenter > Menu > Workload Management
  - Click Enable, and you should see the clusters in the comptible list
  - Give the network details in there and select the storage policy that we created above
  - Review and apply

* To view the logs of the ongoing activities:

> ``` 
> ssh root@vcenter.homelab.io
> shell
> tail -f /var/log/vmware/wcp/wcpsvc.logtail -f /var/log/vmware/> wcp/wcpsvc.log
> ```