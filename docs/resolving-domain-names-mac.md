Resolving domain names on Mac
---

Having trouble resolving domain names on your mac??

Here is the quickest solution. Mac allows us to specify resolvers, that map to a domain and have a `nameserver` associated with it. Once the network is restarted, we can then resolve machines with their domain names.

Follow these steps:

- Create the `/etc/resolver` directory `sudo mkdir -p /etc/resolver`
- Next create your domain file and add the nameserver with its ip into it:
- `sudo nano /etc/resolver/test.com` and add
  > `nameserver 172.80.10.2`

- Save the file
- Restart the network
  - `sudo ifconfig en0 down`, assuming you are using `en0` for gaining your IP address
  - `sudo ifconfig en0 up`

Now ping the machine using the A record mentioned in your DNS, and success!!
