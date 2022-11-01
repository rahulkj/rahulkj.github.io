---
layout: post
title:  "Troubleshoot the workload clusters"
date:   2022-11-01 17:23:00 +0530
categories: tanzu, kubernetes, workload clusters, troubleshooting
---

You might have a need to login to your workload clusters to troubleshoot some issues related to:
* DNS
* Certificates
* External connectivity, etc

So how do we do this?

### Troubleshooting by connecting to the vCenter - Non routable workload cluster IP's

- You can ssh into your vCenter using the **root** crdentials:

    `ssh root@vcenter.example.com -o PubkeyAuthentication=no`

- Once in, switch the mode to `shell` mode.
    ```
    Command> shell
    Shell access is granted to root
    root@vcenter [ ~ ]#
    ```

- Now let's get the IP and password to login to the Supervisior Cluster Node. To do this, run the following:

    `/usr/lib/vmware-wcp/decryptK8Pwd.py`

    which will result with an output like the following:

    ```
    root@vcenter [ ~ ]# /usr/lib/vmware-wcp/decryptK8Pwd.py
    Read key from file

    Connected to PSQL

    Cluster: domain-c47:41454ea7-90e7-4c4f-8d79-0fa666784761
    IP: 10.16.0.16
    PWD: FObx+Q06okhCYjcQvOXy2CgVydgXEyc7dBf6X18zhQLBddrvex4SbUAgXDdCTdxtFRXoooOO/U4Nd9wMGy6qklhWPLWnnZ4CAKCCUfr27SPbRfEKOFA0f46RbDYc2lLDaIsgqOJB/kL/CTNMN5HG/Xw3qWKP3owySsSsSD4Po8=
    ------------------------------------------------------------
    ```

- Using the IP and password combination, let's login to the supervisior cluster using the **root** user. Note, don't decode the password, use it as is.

    `ssh root@10.16.0.16`

    which will give you output similar to below:

    ```
    root@vcenter [ ~ ]# ssh root@10.16.0.16
    FIPS mode initialized
    Password:
    Last login: Tue Nov  1 21:06:25 2022 from 10.16.0.10
    22:33:00 up  2:40,  0 users,  load average: 0.13, 0.34, 0.42
    tdnf update info not available yet!
    ```

- Let's now load the KUBECONFIG to connect to the namespace for the workload cluster. To do this, execute:

    `export KUBECONFIG=/etc/kubernetes/admin.conf`

- Next grab the ssh password to connect to the workload cluster

    `kubectl get secret -n <namespace> <workload-cluster-name>-ssh-password -o jsonpath={.data.ssh-passwordkey} | base64 --decode`

- Now we have all the information to login to thr workload cluster nodes. 

    `ssh vmware-system-user@<workload-cluster-node-ip>`

Now you are in, and you can troubleshoot the problem.


### Troubleshooting by connecting to the Routable workload cluster IP's

If your workload clusters are on routable IP's then you can do this.

- Connect to the cluster using kubectl cli
    `k vsphere login --server api.tkg.example.com --insecure-skip-tls-verify --vsphere-username administrator@vsphere.local --tanzu-kubernetes-cluster-namespace <workload namespace>`

    `k config use-context <workload namespace>`

- Next grab the ssh password to connect to the workload cluster

    `kubectl get secret -n <namespace> <workload-cluster-name>-ssh-password -o jsonpath={.data.ssh-passwordkey} | base64 --decode`

- Now we have all the information to login to thr workload cluster nodes. 

    `ssh vmware-system-user@<workload-cluster-node-ip>`

Now you are in, and you can troubleshoot the problem.


