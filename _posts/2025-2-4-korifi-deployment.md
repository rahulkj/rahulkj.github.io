---
layout: post
title: "Run Korifi on debian based OS running on Raspberry Pi's / vms"
date: 2025-02-04 9:30:00 +0530
categories: korifi, raspberry-pi, ubuntu
---

### Prerequisites
* Ensure you have a 3 node k8s cluster running on your Raspberry Pi's or virtual machines
* Ensure you have a loadbalancer provisioned, if not then checkout [metallb](https://metallb.universe.tf/installation/)
* Provision a storage class and persistent volume
* Make sure your k8s cluster has sufficient resources, like memory and CPU

### Installation
* Clone [this](https://github.com/rahulkj/korifi) repository `https://github.com/rahulkj/korifi`
* Fill out the information in the [env](https://github.com/rahulkj/korifi/env) file
  
  **NOTE:** Make sure your registry has a public signed certificate. If it does not, then please follow the steps outlined in the [official korifi](https://github.com/cloudfoundry/korifi/blob/main/INSTALL.kind.md) docs
  
* Execute the script to install korifi on your k8s cluster 
    ```
    ./korifi_ops.sh install vms   # If installing on amd64 based architecture
    ```

    ```
    ./korifi_ops.sh install pi   # If installing on arm64 based architecture
    ```

* Once the installation is done, you would see the ingress IP to your cf endpoint and apps.

```
{"ip":"10.16.0.121","ipMode":"VIP"}
```

Optionally you can execute the command to get the ingress ip
```
kubectl get service envoy-korifi -n korifi-gateway -ojsonpath='{.status.loadBalancer.ingress[0]}'

{"ip":"10.16.0.121","ipMode":"VIP"}
```

* Next take this IP, and create 2 DNS records

| DOMAIN | IP |
| -- | -- |
| api.korifi.example.com | 10.16.0.121 |
| *.apps.korifi.example.com | 10.16.0.121 |

* Once done, you are all set to play with [cf](https://github.com/cloudfoundry/cli/releases/latest) cli

### Targeting and deploying apps
* First you need to target to the cf endpoint
  > cf api https://api.korifi.example.com --skip-ssl-validation
* Next authenticate and choose the user `cf-admin`
  ```
  cf login
  
  API endpoint: https://api.korifi.example.com

  1. cf-admin
  2. kubernetes-admin
  ```
* Create org / spces
  ```
  cf create-org learn
  cf create-space -o learn dev
  cf target -o learn -s dev
  ```
* Look at all the buildpacks
  ```
  cf buildpacks

  Getting buildpacks as admin...

  position   name                            stack                        enabled   locked   state   filename
  1          paketo-buildpacks/java          io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/java@18.0.0
  2          paketo-buildpacks/go            io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/go@4.13.5
  3          paketo-buildpacks/nodejs        io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/nodejs@7.2.0
  4          paketo-buildpacks/ruby          io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/ruby@0.47.6
  5          paketo-buildpacks/procfile      io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/procfile@5.10.2
  6          paketo-buildpacks/web-servers   io.buildpacks.stacks.jammy   true      false            paketo-buildpacks/web-servers@1.1.5
  ```

* Now lets deploy a very simple app to see if things are working
  ```
  cd samples/webapp

  cf push webapp -p . -m 256M
  Pushing app webapp to org rj / space dev as admin...
  Packaging files to upload...
  Uploading files...
   1.77 KiB / 1.77 KiB [=======================================================================================================================================] 100.00% 1s
  
  Waiting for API to complete processing files...
  
  Staging app and tracing logs...
     Build reason(s): CONFIG
     CONFIG:
     	+ env:
     	+ - name: VCAP_APPLICATION
     	+   valueFrom:
     	+     secretKeyRef:
     	+       key: VCAP_APPLICATION
     	+       name: 1728cf97-6814-432c-b15f-2effa2619424-vcap-application
     	+ - name: VCAP_SERVICES
     	+   valueFrom:
     	+     secretKeyRef:
     	+       key: VCAP_SERVICES
     	+       name: 1728cf97-6814-432c-b15f-2effa2619424-vcap-services
     	resources: {}
     	- source: {}
     	+ source:
     	+   registry:
     	+     image: harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-packages@sha256:c0e79aafa7fe71e88dca20e4f968e7a67991398e3d494ea4d2694f7bf508f85d
     	+     imagePullSecrets:
     	+     - name: image-registry-credentials
     Loading registry credentials from service account secrets
     Loading secret for "harbor.example.com" from secret "image-registry-credentials" at location "/var/build-secrets/image-registry-credentials"
     Loading cluster credential helpers
     Pulling harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-packages@sha256:c0e79aafa7fe71e88dca20e4f968e7a67991398e3d494ea4d2694f7bf508f85d...
     Successfully pulled harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-packages@sha256:c0e79aafa7fe71e88dca20e4f968e7a67991398e3d494ea4d2694f7bf508f85d in path "/workspace"
     Image with name "harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-droplets" not found
     target distro name/version labels not found, reading /etc/os-release file
     ======== Output: paketo-buildpacks/node-start@2.1.18 ========
     could not find app in /workspace: expected one of server.js | server.cjs | server.mjs | app.js | app.cjs | app.mjs | main.js | main.cjs | main.mjs | index.js | index.cjs | index.mjs
     err:  paketo-buildpacks/node-start@2.1.18 (1)
     ======== Output: paketo-buildpacks/node-start@2.1.18 ========
     could not find app in /workspace: expected one of server.js | server.cjs | server.mjs | app.js | app.cjs | app.mjs | main.js | main.cjs | main.mjs | index.js | index.cjs | index.mjs
     err:  paketo-buildpacks/node-start@2.1.18 (1)
     ======== Output: paketo-buildpacks/node-start@2.1.18 ========
     could not find app in /workspace: expected one of server.js | server.cjs | server.mjs | app.js | app.cjs | app.mjs | main.js | main.cjs | main.mjs | index.js | index.cjs | index.mjs
     err:  paketo-buildpacks/node-start@2.1.18 (1)
     3 of 7 buildpacks participating
     paketo-buildpacks/ca-certificates 3.9.0
     paketo-buildpacks/httpd           0.7.39
     paketo-buildpacks/source-removal  0.2.26
     Warning: No cached data will be used, no cache specified.
     target distro name/version labels not found, reading /etc/os-release file
  
     Paketo Buildpack for CA Certificates 3.9.0
       https://github.com/paketo-buildpacks/ca-certificates
       Build Configuration:
         $BP_EMBED_CERTS                    false  Embed certificates into the image
         $BP_ENABLE_RUNTIME_CERT_BINDING    true   Deprecated: Enable/disable certificate helper layer to add certs at runtime
         $BP_RUNTIME_CERT_BINDING_DISABLED  false  Disable certificate helper layer to add certs at runtime
       Launch Helper: Contributing to layer
         Creating /layers/paketo-buildpacks_ca-certificates/helper/exec.d/ca-certificates-helper
     Paketo Buildpack for Apache HTTP Server 0.7.39
       Resolving Apache HTTP Server version
         Candidate version sources (in priority order):
           <unknown> -> "*"
  
         Selected Apache HTTP Server version (using <unknown>): 2.4.62
  
       Executing build process
         Installing Apache HTTP Server 2.4.62
           Completed in 850ms
  
       Configuring launch environment
         APP_ROOT    -> "/workspace"
         SERVER_ROOT -> "/layers/paketo-buildpacks_httpd/httpd"
  
       Assigning launch processes:
         web (default): httpd -f /workspace/httpd.conf -k start -DFOREGROUND
  
       Generating SBOM for /layers/paketo-buildpacks_httpd/httpd
           Completed in 0s
  
     Warning: No cached data will be used, no cache specified.
     Adding layer 'paketo-buildpacks/ca-certificates:helper'
     Adding layer 'paketo-buildpacks/httpd:httpd'
     Adding layer 'buildpacksio/lifecycle:launch.sbom'
     Added 1/1 app layer(s)
     Adding layer 'buildpacksio/lifecycle:launcher'
     Adding layer 'buildpacksio/lifecycle:config'
     Adding layer 'buildpacksio/lifecycle:process-types'
     Adding label 'io.buildpacks.lifecycle.metadata'
     Adding label 'io.buildpacks.build.metadata'
     Adding label 'io.buildpacks.project.metadata'
     Setting default process type 'web'
     Saving harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-droplets...
     *** Images (sha256:b888dd4c9253a7b0bf2154bef108f0a331eb65276154ec5cd3cd6abe2171ef69):
           harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-droplets
           harbor.example.com/korifi/1728cf97-6814-432c-b15f-2effa2619424-droplets:b1.20250204.162654
     Build successful
  
  Waiting for app webapp to start...
  
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  Instances starting...
  
  name:              webapp
  requested state:   started
  routes:            webapp.apps.example.com
  last uploaded:     Tue 04 Feb 10:26:53 CST 2025
  stack:             io.buildpacks.stacks.jammy
  buildpacks:
  
  type:            web
  sidecars:
  instances:       1/1
  memory usage:    256M
  start command:   httpd "-f" "/workspace/httpd.conf" "-k" "start" "-DFOREGROUND"
       state     since                  cpu    memory     disk       logging        cpu entitlement   details
  #0   running   2025-02-04T16:28:55Z   0.0%   0B of 0B   0B of 0B   0B/s of 0B/s

  ```

* Now that the app deployed fine, lets access it in the browser `http://webapp.apps.example.com` to see the message `My first webapp on korifi!`

Now deploy more apps, and please share your learnings via PR's.

Cheers!