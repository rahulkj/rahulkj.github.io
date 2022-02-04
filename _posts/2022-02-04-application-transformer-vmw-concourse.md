---
layout: post
title:  "CLI for Automating the configuration of Application Transformer for VMware Tanzu"
date:   2022-02-04 16:51:00 +0530
categories: application-transformer, application, transformer, vmware, tanzu, at-cli
---

If you've read the previous [blog post](./2022-02-04-application-transformer-vmw.md), then you would have noticed there are a lot of manual steps involved in deployment of the ATVT appliance and configuring it.

The good news is we can automate most of it. There are tools like govc, and ATVT provides api's that can be used to script all the UI actions.

Since you can find a lot of articles on how to import the OVA using govc, lets directly jump into the [tanzu-apptx-cli](https://github.com/rahulkj/tanzu-apptx-cli/releases)

The CLI is written in go and have the MVP features built to configure vCenters, credentials, vRNI, etc.

Here's the help output from the cli:

```
./tanzu-apptx-cli

tanzu-apptx-cli version: 'v1.0.0-dev'
Usage: 'tanzu-apptx-cli [command]'

Available Commands:
  service-account 		Service Accounts operations
  global-default 		Global Defaults operations
  vcenter 			vCenter operations
  vrni 				vRNI operations
  virtual-machines 		Virtual Machines operations
  components 			Components operations
  applications 			Applications operations

```

Before we begin, lets export the common variables that are needed for all the commands.

```
export ATVT_FQDN=10.0.0.10
export ATVT_USER=admin
export ATVT_PASSWORD=Password1234%
```

**NOTE: The values provided here are dummy values, please them those according to your ATVT OVA values**

### Configure Credentials (Service Accounts) in ATVT

* The workflow of using the cli, would begin with configuring the service-account/s. These are accounts that provide us access to vCenter, vRNI, VMs, Registry, etc. To configure the service-account

  ```
  ./tanzu-apptx-cli service-account register -fqdn "${ATVT_FQDN}" -username "${ATVT_USER}" -password "${ATVT_PASSWORD}" -sa-alias vCenter -service-username 'administrator@vsphere.local' -service-password 'Some-Password'


  tanzu-apptx-cli version: 'v1.0.0-dev'
  Service Account created
  ```

* If the credential already exists, the cli would exit with code 0, with the output similar to this

  ```
  tanzu-apptx-cli version: 'v1.0.0-dev'
  Service Account already exists
  ```

Repeat the process to register all the service accounts


### Configure Global Defaults

* This is a very useful feature where we can set the global defaults for allowing ATVT to use the right credentials for the various operations being peformed on various entities. To set the global defaults, run:

  ```
  ./tanzu-apptx-cli global-default assign
  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli global-default assign [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -password string
        Application Transformer admin password
    -sa-alias string
        service account alias
    -service-account-type string
        service account type, ex: VCs, VRNIs, LINUX_VMs <--- ENUM / CONSTANTS
    -username string
        Application Transformer admin username
  ```


### Configure vCenters in ATVT

* Once the service accounts are loaded in the system, you can register the vCenter/s with ATVT. To do this, execute

  ```
  ./tanzu-apptx-cli vcenter register register -fqdn "${ATVT_FQDN}" -username "${ATVT_USER}" -password "${ATVT_PASSWORD}" -sa-alias vCenter -vc-fqdn <VCENTER_IP> -vc-name vCenter-<Identifier>
  ```

* There are check in place to see if the service account exists, or if the vCenter has already been registered. You would see the error output like this

  ```
  tanzu-apptx-cli version: 'v1.0.0-dev'
  Cannot complete the operation as the Service Account does not exist
  Failed to register vCenter with the provided information. Response Code: 400
  ```

### Configure vRNI (VRealize Network Insight)

* If you have a vRNI instance that is monitoring the vCenter that is configured with ATVT, you can register that vRNI instance by running the following command:

  ```
  ./tanzu-apptx-cli vrni register

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli vrni register [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -isSaaS
        using a SaaS vRNI instance, default is false
    -password string
        Application Transformer admin password
    -sa-alias string
        vRNI service account alias
    -username string
        Application Transformer admin username
    -vc-names string
        comma separated list of vCenter Name(s)
    -vrni-api-token string
        SaaS vRNI api token
    -vrni-fqdn string
        vCenter FQDN
  ```

Supply the `-isSaas` flag and `-vrni-api-token` only when the vRNI instance being used here is the SaaS version, else you can ignore these flags, to register the vRNI that is on-prem

### Discovering the Virtual Machines

* Now that we are done with configuring the required entities, we can get to the meat of the product. We first would like to discover all the virtual machines that are managed by vCenter. To do this, execute

  ```
  ./tanzu-apptx-cli vcenter scan-virtual-machines

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli vcenter scan-virtual-machines [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
    -vc-fqdn string
        vCenter FQDN
    -vc-name string
        vCenter Name
  ```

### Discovery of components

* Once the Virtual machines are discovered, we can fire the next command to identify the components that are running on the VMs

  ```
  ./tanzu-apptx-cli vcenter scan-components

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli vcenter scan-components [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
    -vc-fqdn string
        vCenter FQDN
    -vc-name string
        vCenter Name
  ```

### Using vRNI data to discover logical Applications

* One powerful feature that ATVT brings to the table, is topology discovery, where ATVT connects all the components that are communicating with each other, to then logically group them into an application. TO run this process, execute:
  
  ```
  ./tanzu-apptx-cli vcenter discover-topology

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli vcenter discover-topology [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
    -vc-fqdn string
        vCenter FQDN
    -vc-name string
        vCenter Name
  ```

### Extracting Reports

* We can download the reports in cvs,json,table format from ATVT, for virtual machines, components and applications. We can use the following commands:



* List Components
  ```
  ./tanzu-apptx-cli components list

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli components list [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -output-format string
        Output format - (json,csv,table) (Default: table) (default "table")
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
  ```

* List Applications

  ```
  ./tanzu-apptx-cli applications list

  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli applications list [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -output-format string
        Output format - json,csv,table (default "table")
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
  ```

* List Virtual Machines
  ```
  ./tanzu-apptx-cli virtual-machines list
  tanzu-apptx-cli version: 'v1.0.0-dev'
  Usage: 'tanzu-apptx-cli virtual-machines list [flags]'
  Available Flags:
    -fqdn string
        Application Transformer FQDN / IP, ex: appliance.example.com
    -output-format string
        Output format - (json,csv,table) (Default: table) (default "table")
    -password string
        Application Transformer admin password
    -username string
        Application Transformer admin username
    -vc-cluster string
        vCenter Cluster Name
    -vc-datacenter string
        vCenter Datacenter
    -vc-folder string
        vCenter Folder Name
    -vc-fqdn string
        vCenter FQDN
    -vc-resource-pool string
        vCenter Resource Pool Name
    -vm-ip string
        Virtual Machine IP
    -vm-name string
        Virtual Machine Name
  ```

More to come, feel free to leave your feedback on [Github](https://github.com/rahulkj/tanzu-apptx-cli)