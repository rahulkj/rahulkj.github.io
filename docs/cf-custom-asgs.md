Custom ASGs for Cloud Foundry
---

- Execute `cf security-groups`. The output will look like this

  ```
  > cf security-groups
  Getting security groups as admin...
  OK
  name organization space lifecycle
  #0 default_security_group running
  default_security_group staging
  #1 healthwatch_security_group system healthwatch running
  #2 sql_open system nfs running
  ```

- To restrict the security group, modify the `default_security_group` to include just the deployment and services network, for ex:

- Execute: `cf security-group default_security_group > default_security_group.json`
- modify the `default_security_group.json` to look like
```
[
  {
    "description": "Deployment network",
    "destination": "172.16.20.0/22",
    "protocol": "all"
  },
  {
    "description": "Services network",
    "destination": "172.16.22.0/22",
    "protocol": "all"
  }
]
```

- Create a new security group by executing: `cf security-group custom_running_security_group default_security_group.json`

- Unbind the `default_security_group` and disable the running lifecycle security group. `cf unbind-running-security-group default_security_group`

- The output for `cf security-groups` should look like this after running the above command

  ```
  > cf security-groups
  Getting security groups as admin...
  OK
  name organization space lifecycle
  #0 default_security_group staging
  #1 healthwatch_security_group system healthwatch running
  #2 sql_open system nfs running
  #3 custom_running_security_group
  ```

- Now, bind the `custom_running_security_group` to all orgs in the platform, by executing `cf bind-running-security-group custom_running_security_group`

- The output for `cf security-groups` should look like this after running the above command

  ```
  > cf security-groups
  Getting security groups as admin...
  OK
  name organization space lifecycle
  #0 default_security_group staging
  #1 healthwatch_security_group system healthwatch running
  #2 sql_open system nfs running
  #3 custom_running_security_group running
  ```
