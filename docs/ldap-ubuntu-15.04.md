Setup LDAP on Ubuntu 15.04
---

Its pretty straight forward to setup ldap on `Ubuntu 15.04`

- Set your `hostname` and update `/etc/hosts` file
- `sudo apt-get install slapd ldap-utils`
- Execute `ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn` and you can find the admin account
- Import your `ldif` using `ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_content.ldif`

Done!!

I wrote this for my reference, if you find it helpful, then please leave your comments

For complete details, here is the official site: https://help.ubuntu.com/lts/serverguide/openldap-server.html
