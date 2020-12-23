---
layout: post
title:  "vSphere 7 + Workload Management + HAProxy"
date:   2020-11-18 5:00:00 -0600
categories: tkg, vsphere 7, workload management, homelab, haproxy
---

To enable Workload Management on vSphere 7 and using HAProxy as the networking stack, ensure the following steps are completed:

* Create a private network port group in vSphere, and attach to `vmnic1`
* Setup HAProxy appliance by downloading, and import the OVA from https://github.com/haproxytech/vmware-haproxy
  - Attach the Management Network and Frontend Network to a routable network
  - Attaching Workload Network to the port group associate with the private network
  - Allocate a `/28` CIDR block for Frontend/Load Balancer IPs

  | Component | IP Address/CIDR |
  | -- | -- |
  | Management IP | 10.0.0.44 |
  | Workload IP | 192.168.10.10 |
  | Load Balancer Service IP Range | 10.0.0.144/28 |
  | Supervisior Cluster IP Range | 10.0.0.160/28 |

* Create a Tag on the storage/s that will be used for workload management
  - Click in vCenter > Storage
  - Select the storage and click on Actions > Tags & Custom Attributes > Assign Tag
  - Create a tag with the name `k8s` and Category name as `k8s` and select all
  - Assign the tag
* Create a `VM Storage Policies` by clicking in vCenter > `Policies and Profiles` > `VM Storage Policies`
  - Click on `Create VM Storage Policy`
  - Name: `k8s-storage`
  - Enable tag based placement rules
  - Tag Category: `k8s` and Tags `k8s`
  - Storage compatibility: should see all the storages
* On the cluster, enable DRS and HA. Ensure the vmk0 is enabled for vMotion & vSAN

* Download and host the content library on linux box script
  ```
  #!/bin/bash -x

  CONTENT_URL=https://wp-content.vmware.com/v2/latest/
  CONTENT_FOLDER=/data/vmware/content

  mkdir ${CONTENT_FOLDER}
  wget ${CONTENT_URL}/items.json -O ${CONTENT_FOLDER}/items.json
  wget ${CONTENT_URL}/lib.json -O ${CONTENT_FOLDER}/lib.json

  FOLDERS=$(cat ${CONTENT_FOLDER}/items.json | jq -r '.items[] | .name')

  for f in ${FOLDERS}; do
    if [[ ! -d "${CONTENT_FOLDER}/${f}" ]]; then
      mkdir -p ${CONTENT_FOLDER}/${f}
    fi

    pushd ${CONTENT_FOLDER}/${f}
      if [[ ! -f "item.json" ]]; then
        wget ${CONTENT_URL}/$f/item.json -O item.json
      fi
      FILES=$(cat item.json | jq -r '.files[] | .name')
      for file in ${FILES}; do
        if [[ ! -f "${file}" ]]; then
          wget ${CONTENT_URL}/$f/$file -O $file
        fi
      done
    popd
  done
  ```
* Install nginx on `ubuntu.homelab.io` and set the port to listen on 9090, update the /etc/nginx/sites-available/default
  ```
  server {
    listen 9090 default_server;
    listen [::]:9090 default_server;

    root /data;

    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
      autoindex on;
      try_files $uri $uri/ =404;
    }
  }
  ```
* Setup content Library by clicking on vCenter > Menu > Content Libraries
  - Create one with the name `k8s` and  Subscribed content library: `http://ubuntu.homelab.io:9090/vmware/content/lib.json`

* If all the above is done, then click on vCenter > Menu > Workload Management
  - Click Enable, and you should see the clusters in the compatible list
    - vCenter Server and Network: `vCenter Server Network`
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-1.png)

    - Cluster: `WORKLOAD`
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-2.png)

    - Size: `Tiny`
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-3.png)

    - Storage:
      - Control Plane Node: `k8s-storage`

      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-4.png)

    - Load Balancer:
      - Name: `haproxy`
      - Type: `HA Proxy`
      - Data plane API Address: `10.0.0.44:5556`
      - User name: `admin`
      - Password: `password`
      - IP Address Ranges for Virtual Servers: `10.0.0.144-10.0.0.159`
      - Server Certificate: Get the value by running the command > `echo | openssl s_client -showcerts -connect 10.0.0.143:5556 2>/dev/null | openssl x509 -inform pem -text`
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-5.png)

    - Management Network:
      - Network: `EDGE-UPLINK-PG`
      - Start IP Address: `10.0.0.160`
      - Subnet Mask: `255.255.252.0`
      - Gateway: `10.0.0.1`
      - DNS Server: `10.0.0.11`
      - DNS Search Domain: `homelab.io`
      - NTP Server: `10.0.0.12`
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-6.png)

    - Workload Network:
      - IP address for Services: `10.96.0.0/24`
      - DNS Servers: `10.0.0.11`
      - Workload Network:
        - Name: `workload-network`
        - Prot Group: `TKG`
        - Gateway: `192.168.10.1`
        - Subnet: `255.255.254.0`
        - IP Address Ranges: `192.168.10.20-192.168.10.255`

      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-7.png)

      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-8.png)

    - TKG Configuration:
      - Content Library: `k8s`
  
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-9.png)

    - Review and apply
      
      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-10.png)

      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-11.png)

      ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-12.png)

* To view the logs of the ongoing activities:

> ``` 
> ssh root@vcenter.homelab.io
> shell
> tail -f /var/log/vmware/wcp/wcpsvc.logtail -f /var/log/vmware/> wcp/wcpsvc.log
> ```

* Create a namespace `k8s1` using the Workload Management

  ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-14.png)

  ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-15.png)

  ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-16.png)

* Download the kubectl vsphere plugin, by connecting to the supervisor cluster. Click on the `k8s1` and on summary > status, click on `Open` link to CLI tools. Download the cli and put it in the path

  ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-13.png)

* Next connect to the supervisor cluster `k vsphere login --server=https://10.0.0.144/ --insecure-skip-tls-verify --vsphere-username=administrator@homelab.io` and login as administrator
`k config use-context k8s1`

* List all the k8s versions available
  ```
  k get virtualmachineimages

  NAME                                                         VERSION                           OSTYPE
  ob-15957779-photon-3-k8s-v1.16.8---vmware.1-tkg.3.60d2ffd    v1.16.8+vmware.1-tkg.3.60d2ffd    vmwarePhoton64Guest
  ob-16466772-photon-3-k8s-v1.17.7---vmware.1-tkg.1.154236c    v1.17.7+vmware.1-tkg.1.154236c    vmwarePhoton64Guest
  ob-16545581-photon-3-k8s-v1.16.12---vmware.1-tkg.1.da7afe7   v1.16.12+vmware.1-tkg.1.da7afe7   vmwarePhoton64Guest
  ob-16551547-photon-3-k8s-v1.17.8---vmware.1-tkg.1.5417466    v1.17.8+vmware.1-tkg.1.5417466    vmwarePhoton64Guest
  ob-16897056-photon-3-k8s-v1.16.14---vmware.1-tkg.1.ada4837   v1.16.14+vmware.1-tkg.1.ada4837   vmwarePhoton64Guest
  ob-16924026-photon-3-k8s-v1.18.5---vmware.1-tkg.1.c40d30d    v1.18.5+vmware.1-tkg.1.c40d30d    vmwarePhoton64Guest
  ob-16924027-photon-3-k8s-v1.17.11---vmware.1-tkg.1.15f1e18   v1.17.11+vmware.1-tkg.1.15f1e18   vmwarePhoton64Guest
  ob-17010758-photon-3-k8s-v1.17.11---vmware.1-tkg.2.ad3d374   v1.17.11+vmware.1-tkg.2.ad3d374   vmwarePhoton64Guest
  ```

* Create a cluster yaml and then run `k create -f cluster.yaml`
  ```
  apiVersion: run.tanzu.vmware.com/v1alpha1
  kind: TanzuKubernetesCluster
  metadata:
    name: k8s1-cluster-1
    namespace: k8s1
  spec:
    topology:
      controlPlane:
        count: 3
        class: best-effort-xsmall # https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-7351EEFF-4EF0-468F-A19B-6CEA40983D3D.html
        storageClass: k8s-storage
      workers:
        count: 3
        class: best-effort-xsmall # https://docs.vmware.com/en/VMware-vSphere/7.0/vmware-vsphere-with-tanzu/GUID-7351EEFF-4EF0-468F-A19B-6CEA40983D3D.html
        storageClass: k8s-storage
    distribution:
      version: v1.18.5
    settings:
      network:
        cni:
            name: antrea
        services:
          cidrBlocks: ["198.51.100.0/12"]
        pods:
          cidrBlocks: ["192.0.2.0/16"]
  ```

  ```
  k apply -f cluster.yaml

  tanzukubernetescluster.run.tanzu.vmware.com/k8s1-cluster created
  ```

  This will take a while, and you can watch the cluster creation by `k get TanzuKubernetesCluster -w`

* After the cluster creation is complete, you should see
  ```
  k get TanzuKubernetesCluster
  NAME           CONTROL PLANE   WORKER   DISTRIBUTION                     AGE     PHASE
  k8s1-cluster-1   3               3        v1.18.5+vmware.1-tkg.1.c40d30d   9m24s   running
  ```

  ![]({{ site.url }}/assets/v7-k8s-haproxy/k8s-haproxy-17.png)

* Finally `k vsphere login --server=https://10.0.0.144/ --insecure-skip-tls-verify --vsphere-username=administrator@homelab.io --tanzu-kubernetes-cluster-namespace=k8s1 --tanzu-kubernetes-cluster-name=k8s1-cluster-1`