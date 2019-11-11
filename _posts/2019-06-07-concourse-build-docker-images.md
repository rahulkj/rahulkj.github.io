---
layout: post
title:  "Build and upload docker images using concourse"
date:   2019-06-07 21:15:00 -0600
categories: concourse docker
---

Concourse is an amazing pipelining tool. Given the extensible nature of the tool, one can write a pipeline to execute anything. The fun never ends here.

If you wish to write a pipeline, that needs to create a docker image and host it on your or public docker repository, then check this out.

Define a `pipeline.yml` with the following contents:
```
---
jobs:
- name: build-docker-image
  public: true
  serial: true
  plan:
  - get: schedule
    trigger: true
  - get: git-repo
  - put: docker-image
    params:
      build: git-repo/ci

resources:
- name: schedule
  type: time
  source:
    interval: 24h
    start: "12:00 AM"
    stop: "11:59 PM"
    location: America/Los_Angeles
    days: [Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday]

- name: git-repo
  type: git
  source:
    uri: ((git_url))
    branch: ((git_branch))
    username: ((git_username))
    password: ((git_token))

- name: docker-image
  type: docker-image
  source:
    email: ((docker_hub_email))
    username: ((docker_hub_username))
    password: ((docker_hub_password))
    repository: ((docker_hub_repository))
```

Define a `params.yml` file with the following contents:
```
git_url:   # Your git url
git_branch:  # Your git branch
git_username:  # Your git username
git_token:  # Your git token
docker_hub_email:  # Your docker email
docker_hub_username:  # Your docker username
docker_hub_password:  # Your docker password (ofcourse)
docker_hub_repository:  # Your docker repository
```

Specify the details and finally `fly` the pipeline by executing:
`fly -t <YOUR-ALIAS> sp -p docker-image -c pipeline.yml -l params.yml`

Before you run the pipeline, you need to ensure that your `git` repository has the `Dockerfile` placed in the `ci` folder:
```
.
 ├── ci
 │   ├── Dockerfile
 │   └── some-script.sh
 ```

 Commit your code, and finally run the pipeline. This pipeline would run once daily, and should push out the new docker images with the defined packages to your docker repository.

 Hope this helped you! Cheers~
