---
layout: post
title:  "VMware Avi Deployment and integration with k8s clusters"
date:   2020-12-11 14:00:00 -0600
categories: avi, k8s, vmware
---

Plan your deployment topology:

| Network | VM Network | Subnet | IP Address Pool | Gateway |
| --- | --- | --- | --- | --- |
| AVI Controller | EGDE-UPLINK-PG | 10.0.0.0/22 | 10.0.0.50 | 10.0.0.1 |
| Management Network | EGDE-UPLINK-PG | 10.0.0.0/22 | 10.0.0.161-10.0.0.190 | 10.0.0.1 |

* Download and import the OVA using `govc` if you want to use the cli

  ```
  {
    "DiskProvisioning": "flat",
    "IPAllocationPolicy": "dhcpPolicy",
    "IPProtocol": "IPv4",
    "PropertyMapping": [
      {
        "Key": "avi.mgmt-ip.CONTROLLER",
        "Value": "10.0.0.50"          <----- CHANGE THIS
      },
      {
        "Key": "avi.mgmt-mask.CONTROLLER",
      "Value": "255.255.252.0"        <----- CHANGE THIS
      },
      {
        "Key": "avi.default-gw.CONTROLLER",
        "Value": "10.0.0.1"           <----- CHANGE THIS
      },
      {
        "Key": "avi.sysadmin-public-key.CONTROLLER",
        "Value": ""
      }
    ],
    "NetworkMapping": [
      {
        "Name": "Management",
        "Network": "EDGE-UPLINK-PG"   <----- CHANGE THIS
      }
    ],
    "MarkAsTemplate": false,
    "PowerOn": true,
    "InjectOvfEnv": false,
    "WaitForIP": false,
    "Name": "avi-controller"          <----- CHANGE THIS
  }
  ```

  Store the file as `avi.json`

  `govc import.ova -options=avi.json controller-20.1.2-9171.ova`

* Once the VM is running in vCenter, access the AVI Controller VM IP in the browser `https://10.0.0.50`
  
* Create the Administrator Account
  ![]({{ site.url }}/assets/avi/avi-1.png)

* Configure the system settings (DNS/NTP & Backup passphrase)
  ![]({{ site.url }}/assets/avi/avi-2.png)

* Optionally configure the Email/SMTP
  ![]({{ site.url }}/assets/avi/avi-3.png)

* Select vCenter as the Orchestrator Integration
  ![]({{ site.url }}/assets/avi/avi-4.png)

* Specify the vCenter details with `Write` permissions. Skip SDN Integration
  ![]({{ site.url }}/assets/avi/avi-5.png)

* Select the datacenter
  ![]({{ site.url }}/assets/avi/avi-6.png)

* Select the Management network and define the IP Pool
  ![]({{ site.url }}/assets/avi/avi-7.png)

* Select `NO` in tenant settings
  ![]({{ site.url }}/assets/avi/avi-8.png)

* That's the initial setup
  ![]({{ site.url }}/assets/avi/avi-9.png)

* Click on `Templates` in the top left beside `Applications`

* Go into IPAM/DNS Profiles and create the following:
  - IPAM Profile
    - Name: k8s-ipam-profile
    - Type: Avi Vantage IPAM
    - Allocate IP in VRF is `Checked`
    - Avi Vantage IPAM Configuration
      - Cloud for Usable Network: `Default-Cloud`
      - Usable Network: `EDGE-UPLINK-PG`
  - DNS Profile
    - Name: k8s-dns-profile
    - Type: Avi Vantage DNS
    - Avi Vantage DNS Configuration
      - Default Record TTL for all domains: `30`
      - Domain Name:
        - `avi.k8s1.pks.homelab.io`
  
  ![]({{ site.url }}/assets/avi/avi-10.png)

* Next associate the DNS and IPAM profiles to the `Default-Cloud`, which is under Infrastructure > Clouds

  ![]({{ site.url }}/assets/avi/avi-11.png)

* Follow this to run the helm charts on the k8s cluster
  - https://avinetworks.github.io/avi-helm-charts/docs/AKO/

* `helm repo add ako https://avinetworks.github.io/avi-helm-charts/charts/stable/ako`

* Download and modify the ako values.yml from [here](https://github.com/avinetworks/avi-helm-charts/blob/master/charts/stable/ako/values.yaml)

* `helm install ako/ako --generate-name --version 1.2.1 -f values.yaml -n avi-system`

* Deploy a test workload on the k8s cluster
  * `k run test-app --image=rjain/sample-k8s-app --port=8080`
  * `k expose pod test --port 80 --target-port 8080 --type=LoadBalancer`

* If everything is working, then you should see an external IP assigned to the svc, when you run `k get svc`

That's all!!