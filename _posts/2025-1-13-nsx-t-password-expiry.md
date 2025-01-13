---
layout: post
title: "NSX-T Password expiry"
date: 2025-01-13 15:30:00 +0530
categories: nsx-t
---

Please use this article only when operating NSX-T in your homelab, not recommended for production use.

Offlate, I started getting notifications from my NSX Manager console, that my passwords for root, admin, audit were going to expire.

![]({{ site.url }}/assets/nsx-t/NSX-T-Alarms.png

While operating in my lab environment, I would prefer to use a strong password, but something that wouldn't expire, as it takes a lot of effort to update the secrets in all the services.

### Option 1: Using NSX Manager console (UI)
To remediate this, the recommended approach is to change the password, by navigating to System > User Management > Local Users > Select the 3 dots by the user > Change Password

![]({{ site.url }}/assets/nsx-t/NSX-T-Change_Password.png

Once done for one user, repeat the same steps for the rest of users, and now you are golden.

### Option 2: Using nsxcli

Another approach is to do this using the nsxcli, but ssh'ing onto your NSX Manager virtual machine

- Use a secure program to connect to the NSX CLI console.
- Reset the expiration period, by setting the expiration period between 1 and 9999 days.
> nsxcli> set user admin password-expiration <1 - 9999>

- Or, You can disable password expiry so the password never expires.
> nsxcli> clear user admin password-expiration

I went with Option 2, where I disabled the password expiration for all the internal users.

Hope this helps someone. Cheers!