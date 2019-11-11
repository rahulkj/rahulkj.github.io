---
layout: post
title:  "Session Replication with an External Gemfire Server for Applications deployed within CloudFoundry"
date:   2015-08-19 22:00:00 -0600
categories: session_replication gemfire cloudfoundry cloud_native
---

Did you know CloudFoundry allows us to offload session management to `Gemfire` and `Redis`. as a developer, you've got to just bind to a service, and that's pretty much it, no special coding required. So here's a small write-up to help you on setting up the standalone external `Gemfire` server to deal with sessions.

Firstly gather the following jars:
```
gemfire.jar
gemfire-modules-8.1.0.jar
gemfire-modules-session-8.0.jar
catalina.jar
tomcat-util.jar
tomcat-juli.jar
servlet-api.jar
slf4j-api-1.5.8.jar
slf4j-jdk14-1.5.8.jar
gemfire-security-0.0.1-SNAPSHOT.jar (We'll discuss about this jar a bit later)
```

Now optionally you can create the `gemfire.properties` for your locator:

```
bind-address=LOCATOR IP
log-file=/home/ubuntu/Documents/gemfire/locator1/locator.log
statistic-sampling-enabled=true
log-file-size-limit=50
enable-network-partition-detection=false
statistic-archive-file=/home/ubuntu/Documents/gemfire/locator.gfs
http-service-port=8080
jmx-manager-start=true
archive-disk-space-limit=50
archive-file-size-limit=10
log-disk-space-limit=500
statistic-sample-rate=1000
conserve-sockets=false
mcast-port=0
log-level=fine
```
And for server:

```
log-file=/home/ubuntu/Documents/gemfire/server1/server.log
archive-disk-space-limit=50
conserve-sockets=false
security-client-authenticator=io.pivotal.BasicAuthenticator.create
log-disk-space-limit=500
enable-network-partition-detection=false
locator-wait-time=300
statistic-archive-file=/home/ubuntu/Documents/gemfire/server1/server.gfs
statistic-sampling-enabled=true
statistic-sample-rate=1000
log-level=fine
bind-address=LOCATOR IP/s
mcast-port=0
log-file-size-limit=50
archive-file-size-limit=10
```

Now let's focus on the jar we created: `gemfire-security-0.0.1-SNAPSHOT.jar`

We create a simple java application, that has the class `io.pivotal.BasicAuthenticator`

REPO LOCATION: https://github.com/rahulkj/gemfire-security

The contents of the class are

```
package io.pivotal;

import java.io.Serializable;
import java.security.Principal;
import java.util.Properties;

import com.gemstone.gemfire.LogWriter;
import com.gemstone.gemfire.distributed.DistributedMember;
import com.gemstone.gemfire.security.AuthenticationFailedException;
import com.gemstone.gemfire.security.Authenticator;

public class BasicAuthenticator implements Authenticator {
  private String systemUsername;
  private String systemPassword;
  private LogWriter logger;

  public static Authenticator create() {
    return new BasicAuthenticator();
  }

  public void init(Properties properties, LogWriter logWriter, LogWriter logWriter2) throws AuthenticationFailedException {
    logger = logWriter;
    systemUsername = System.getProperty("security-username");
    systemPassword = System.getProperty("security-password");
  }

  public Principal authenticate(Properties properties, DistributedMember distributedMember) throws AuthenticationFailedException {
    String givenUsername = properties.getProperty("security-username");
    String givenPassword = properties.getProperty("security-password");

    logger.fine("Authenticating against " + givenUsername + "/"
    + givenPassword);

    if (!(systemUsername.equals(givenUsername) && systemPassword
    .equals(givenPassword))) {
      throw new AuthenticationFailedException(
      "Invalid username/password combination given");
    }

    return new BasicPrincipal(givenUsername);
  }

  public void close() {
  }
}
```

```
public class BasicPrincipal implements Principal, Serializable {

  private final String username;

  public BasicPrincipal(final String username) {
    this.username = username;
  }

  public String getName() {
    return username;
  }

  @Override
  public String toString() {
    return new String("BasicPrincipal[username=" + username);
  }
}
```

Compile this java class and generate the jar file, and finally copy this over to the gemfire server.

Use the following commands to start the locators and servers:

```
export JAR_DIR=/home/ubuntu/Documents/

> start locator --name=locator1 --properties-file=$JAR_DIR/gemfire/gemfire_locator.properties

> start server --name=server1 --J=-Dsecurity-username=gemfire --J=-Dsecurity-password=gemfire --classpath=$JAR_DIR/gemfire.jar:$JAR_DIR/gemfire-modules-8.1.0.jar:$JAR_DIR/gemfire-modules-session-8.0.jar:$JAR_DIR/catalina.jar:$JAR_DIR/tomcat-util.jar:$JAR_DIR/tomcat-juli.jar:$JAR_DIR/servlet-api.jar:$JAR_DIR/slf4j-api-1.5.8.jar:$JAR_DIR/slf4j-jdk14-1.5.8.jar:$JAR_DIR/gemfire-security-0.0.1-SNAPSHOT.jar --properties-file=$JAR_DIR/gemfire/gemfire_server.properties

> start server --name=server2 --J=-Dsecurity-username=gemfire --J=-Dsecurity-password=gemfire --classpath=$JAR_DIR/gemfire.jar:$JAR_DIR/gemfire-modules-8.1.0.jar:$JAR_DIR/gemfire-modules-session-8.0.jar:$JAR_DIR/catalina.jar:$JAR_DIR/tomcat-util.jar:$JAR_DIR/tomcat-juli.jar:$JAR_DIR/servlet-api.jar:$JAR_DIR/slf4j-api-1.5.8.jar:$JAR_DIR/slf4j-jdk14-1.5.8.jar:$JAR_DIR/gemfire-security-0.0.1-SNAPSHOT.jar --properties-file=$JAR_DIR/gemfire/gemfire_server.properties --server-port=40401
```
Finally create a user provided service that follows the syntax: `cf cups session_replication -p "locators,username,password"`

Supply the locators ips: `["LOCATOR1[PORT]",["LOCATOR2[PORT]"]`
and the username/password would be gemfire

Bind this service to your application and restage the application.

You now have the sessions offloaded to gemfire server.
