---
layout: post
title: "Video Playback Over NFS: Stuttering/Dropped Frames"
date: 2012-04-04T20:06:00
comments: true
categories:
---
I play video content shared over NFS on various devices throughout my house. Recently, following a server upgrade, the performance of one of these devices dropped significantly, to the point of causing buffer underruns and compromising playback.

After investigating the mount options used on the device (it's a WDLive with custom firmware), I noticed that the NFS share was being mounted and accessed using UDP. You'd figure that dropped packets wouldn't be a concern for a wired connection on a home network, but there are probably other factors at work. As of v3, NFS supports using TCP, and that's exactly what I had to specify. I'm using xmount to mount and access my NFS share, but that's only because I'm using FUSE. Here's what I ended up having to do:

``` bash
xmount <server_ip>:<remote_share_path> <local_share_name> nfs "proto=tcp"
```

That's it! I'll take the slight overhead of TCP in exchange for better reliability any day.
