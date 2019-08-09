Running batch file as a service (Windows XP only)
---

To run a batch file (.bat) as a windows service, you can perform the following

`sc create "service name" binPath= "[path to batch file]" DisplayName= "[Display Name]"`

The above command creates and registers a service with the service name specified earlier.

After the service has been registered, you should start the service. Type the following: `sc start "service name"`

Similarly, to stop the service: `sc stop "service name"`

To delete the service use the command: `sc delete "service name"`

**Note:** There should be a space between the `options=` and `value`

### References:
- http://support.microsoft.com/kb/251192
