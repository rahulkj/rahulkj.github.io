SSH into the warden containers running within a DEA in bosh-lite
---

If you're using bosh-lite and wondering on how you could get into your application container, then follow the steps below:

- Navigate to the bosh-lite directory
- `vagrant ssh`
- `ssh-keygen -t rsa`
- `bosh download manifest cf-warden cf-warden.yml`
- `sudo bosh deployment cf-warden.yml`
- `bosh ssh runner_z1`
   - Enter the password
- `sudo su -`
- Locate the `/var/vcap/data/dea_next/db/instances.json`, and `vi /var/vcap/data/dea_next/db/instances.json`
- Look for the application name, and capture the `warden_handle`
- `/var/vcap/packages/warden/warden/src/wsh/wsh --socket /var/vcap/data/warden/depot/WARDEN-HANDLE/run/wshd.sock --user vcap`
- You should now be in your warden container

`wsh` - Stands for warden shell, using this we can ssh into our warden containers

Cloud Foundry is an amazing platform for developers and operators. Makes life easy.
