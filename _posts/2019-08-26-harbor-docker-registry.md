---
layout: post
title:  "Harbor & Docker Images"
date:   2019-08-26 14:07:00 -0600
categories: docker harbor registry
---

## Why Harbor?
[Harbor](https://goharbor.io/) is an open source cloud native registry that stores, signs, and scans container images for vulnerabilities.

Harbor solves common challenges by delivering trust, compliance, performance, and interoperability. It fills a gap for organizations and applications that cannot use a public or cloud-based registry, or want a consistent experience across clouds.

- After you install Harbor, you will need to deal with certificates.

- There are 2 options,
  - buy a public certificate or use a self-signed certificate. If you have a public certificate, then using Harbor with docker cli, should be straight forward.
  - But if you have a self-signed certificate, then you will need to add the harbor url in docker insecure registry. Its under `preferences > daemon > insecure registries`


- Restart docker.

- Next is to authenticate to harbor using docker cli: `docker login harbor.example.com`

- Using the harbor UI, create a new project, where you would pushing your docker images. Once all this is done, you can enjoy building and pushing images to your private docker registry

- Build a docker image: `docker build . -t harbor.example.com/rjain/buildbox`

- Finally, push the image up: `docker push harbor.example.com/rjain/buildbox:latest`
