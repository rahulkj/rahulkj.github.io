---
layout: post
title:  "Install and Configuration of various softwares"
date:   2012-11-27 16:00:00 -0600
categories: mac softwares
---

I prefer using HomeBrew to brew my Mac. If you don't already have HomeBrew on your Mac, get it by running
`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

### ActiveMQ

- Install on MacOS
http://activemq.apache.org/getting-started.html#GettingStarted-StartingActiveMQ

`brew install apache-activemq`

Add the following to `~/.profile`

```
export PATH=${PATH}:/usr/local/Cellar/activemq/5.7.0/bin:$PATH
```

Run the commands to start/stop activemq:
```
activemq start
activemq stop
```

Access console on: http://localhost:8161/admin/index.jsp

### Sonatype Nexus

```
brew install nexus

nexus start
nexus stop
```

Access console on http://localhost:8081/nexus

### RabbitMQ

```
brew install rabbitmq-server

defaults:
port: 5672
username: guest
password: guest
```

Access console on http://localhost:15672/
