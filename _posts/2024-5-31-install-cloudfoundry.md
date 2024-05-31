---
layout: post
title:  "Tanzu Platform with CloudFoundry Runtime"
date:   2024-05-31 07:40:00 +0530
categories: cloudfoundry, tanzu
---

Tanzu Platform with CloudFoundry Runtime is a great platform to run your cloud native applications. You can run almost any application on this runtime. This platform can be deployed and operated on any Public IaaS, be it AWS, GCP, Azure or On-Prem IaaS namely vSphere, Openstack etc

Cloudfoundry is a platform that has great features:
- Support for applications written in various languages
- Continously monitors the application health, and heals them if the application crashes due to some reasons
- Evenly deploys applications on the vms that run the workloads
- Provides a routable address for the applications once they are deployed on the platform
- Allows applications to bind to services provided by the platform and outside the platform
- Allows developers to ssh into their running applications to troubleshoot the apps
- Provides capability for injecting environment variables and/or secrets for the applications
- Outputs the application logs to external logging solutions, without any or little code changes
- and there are many more...

[BOSH](https://bosh.io/docs/) (BOSH Outer Shell) is the secret sauce that provisions and manages the deployment of vms on the targetted infrastructure. 

Advantages of BOSH are:
- Manages the lifecycle of the vms
  - If the vm crashes, bosh will bring the vm back online on the available compute in that given AZ
  - Provision of vms using deployment yaml files
  - During upgrades, bosh does canary deployments, where it drains one instance of a given job, and then provisions another, to avoid complete outage of the environment
- Allows capability to repave the platform without impacting the business workloads
- VMs are idempotent, if anyone tweaks anything, then the platform operator can repave the deployment to revert the corrupt state of the vms
- Continous monitoring of the vms and processes running on those vms. It healing of process on the vms is done using `monit`

The platform deployment is very simple, and the platform operators can automate the installation of the platform using [concourse](https://concourse-ci.org/)

Everyone organization run applications that are developed in various languages. Cloudfoundry provides support for a large number of languages. These are provided in terms of [buildpacks](https://docs.vmware.com/en/VMware-Tanzu-Application-Service/6.0/tas-for-vms/toc-system-buildpacks-index.html?hWord=N4IghgNiBcIEYFcCWEAmAHMBjA1iAvkA). Here is a list of [buildpacks](https://docs.vmware.com/en/VMware-Tanzu-Application-Service/6.0/tas-for-vms/system-buildpacks.html):
* Binary - Binary source 
* Go - Go code
* HWC - Hosted Web Core API for running .NET Applications on Windows.
* Java - Grails, Play, Spring, or any other JVM-based language or framework
* .NET Core - .NET Core applications
* NGINX - Static websites
* Node.js - Node or JavaScript applications
* PHP - Cake, Symfony, Zend, NGINX, or HTTPD apps
* Python - Django or Flask
* R - R apps
* Ruby - Ruby, JRuby, Rack, Rails, or Sinatra apps
* Staticfile - HTML, CSS, JavaScript, or NGINX apps

The developers develop their application and using the CI tools, they generate the application artifacts, after which they use the [cf](https://docs.cloudfoundry.org/cf-cli/getting-started.html) cli to deploy the application on the platform. The deployment is not hard, its as simple as `cf push <app-name> -p <artifact>`

The platform then detects the application language, then compiles the droplet (OCI image) using the buildpack and the application artifact, and then finally deploys the application and makes a route available for the developer to access their application. Its that easy.

I hope you got an idea of how BOSH makes it easy for your platform operators to manage the platform, and avoid any weekend maintenance work. Also CloudFoundry makes it easy for developers to focus on developing their business logic, and not worrying about generating OCI images, deployment of applications, networks and firewalls. 