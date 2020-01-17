---
layout: post
title:  "Technical references"
date:   2012-02-20 1:53:00 -0600
categories: technology
---

This page lacks information. Please stay tuned on this.

## GUI FREAKS --&gt; CHECK THIS OUT
-	http://nodejs.org/
-	http://documentcloud.github.com/backbone/
-	http://documentcloud.github.com/underscore/
-	http://documentcloud.github.com/underscore/
-	http://handlebarsjs.com/
-	http://net.tutsplus.com/tutorials/javascript-ajax/getting-started-with-backbone-js/
-	http://github.com

### WEB SERVICES
- http://blog.springsource.org/2011/11/09/using-cloud-foundry-services-with-spring-applications-part-3-the-cloud-namespace/
- http://blog.furiousbob.com/2011/12/06/hateoas-restful-services-using-spring-3-1/

### VMWARE CLOUD FOUNDRY SPRING
- http://start.cloudfoundry.com/tools/vmc/vmc-quick-ref.html
- http://static.springsource.org/spring-roo/reference/html/command-index.html

**If you ever end up with the problem where you have to disable security when invoking the REST based webservices from you javascript in chrome, then try the following:**

`open -a Google\ Chrome.app --args --disable-web-security`

`sudo ln -s $ROO_HOME/bin/roo.sh /usr/bin/roo`

Install `macports` from macports.org
Install `xcode` and command line tools for `xcode` from apple site

`sudo port install nodejs`

```
.profile /Users/
export M2_HOME=/Users//ide/apache-maven-3.0.3/
export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Home
export PATH=/opt/local/bin:/opt/local/sbin:$M2_HOME/bin:$PATH
<h6>Clean up old archives</h6>
sudo gem cleanup
sudo port -f uninstall inactive
brew cleanup
```

### Cocoapods
install cocoapods using

```
gem install cocoapods

pod setup
```

if `pod setup` fails, ensure you delete `~/.cocoapods/*` and try again

Burn a DVD using ISO on mac
`hdiutil burn some-image.iso`
