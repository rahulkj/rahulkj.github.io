---
layout: post
title:  "Running batch file as a service (Windows XP only)"
date:   2013-04-03 8:34:00 -0600
categories: ubuntu storage
---

To run a batch file (.bat) as a windows service, you can perform the following

`sc create "service name" binPath= "[path to batch file]" DisplayName= "[Display Name]"`

The above command creates and registers a service with the service name specified earlier.

After the service has been registered, you should start the service. Type the following: `sc start "service name"`

Similarly, to stop the service: `sc stop "service name"`

To delete the service use the command: `sc delete "service name"`

**Note:** There should be a space between the `options=` and `value`

### References:
- [http://support.microsoft.com/kb/251192](http://support.microsoft.com/kb/251192)
