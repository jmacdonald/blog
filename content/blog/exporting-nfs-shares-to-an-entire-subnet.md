---
layout: post
title: "Exporting NFS Shares to an Entire Subnet"
date: 2012-04-08T15:18:00
comments: true
categories: NFS
---
Exporting file system paths as NFS shares is a relatively simple process, especially if you're only doing it for specific IP addresses. Exporting to an entire subnet is slightly trickier, mostly because it can be difficult to find documentation or examples that illustrate exact how to do it.

Here's an example that'll work for a typical home network subnet:

```bash
# /etc/exports
/mnt/Storage -network 192.168.1.0/24
```
Note the _-network_ option, which I needed when specifying a subnet (which is done in CIDR syntax, as per above).
