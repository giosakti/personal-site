---
title: "Clustering in LXD 3.0"
authors: ["gio"]
date: 2018-08-16T20:00:00+07:00
tags:
- container
- lxd
draft: true
---

*I've been experimenting and working on a project using LXD in the past few months and there is an interesting feature that I want to discuss, which is the **clustering** ability that I found in LXD 3.0.*

When I started working on this project, I initially used LXD 2.x because that is what's available by default on Ubuntu 16.04. However, it turns out that my project needs multiple host machines to work together so I was looking at the options. Happily, LXD 3.0 which released on early April has support for clustering and I can now join my host machines together. It is also an LTS release, so it means that it will be supported until June 2023.

## Upgrading

If you're like me and still using Ubuntu 16.04, LXD 3.0 won't be available by default. One of the easiest option to upgrade (that I also use) is to use the snap package manager. LXD 3.0 package also comes with migration tool, which is really useful. But if you're doing the migration on production environment, always be prepared for unexpected things.

Start upgrading by executing the `snap install` command.

```shell
sudo snap install lxd --channel=3.0
```

Do the migration, extra nice because LXD will ask if we want to remove the old LXD version so that it doesn't conflict.

```shell
```

Make sure that you have the 3.0 version by typing `lxd -v`. Also because we want to try out the clustering mode, we have to provision another machine which also has LXD upgraded to 3.0.

## Bootstrap the first node

Procedure for bootstrapping the first node will be slightly different than all the other node. There are two ways to bootstrap, first is to use the interactive 'wizard' by executing `lxd init` and the other way is to use a 'preseed' file which will be used to setup. We will do the first one in this post, so that we can understand the process more clearly.

## Bootstrap the joining node

On joining node, we can bootstrap the same way by executing `lxd init`, but the answer that we give will be slightly different. Let's try doing it and see what the wizard progression looks like.

## Testing

So now we have 2 host in clustering mode, great! We can try spinning up some containers and see where the containers will end up in.

```shell
sudo lxc launch ubuntu:16.04 test-01
sudo lxc launch ubuntu:16.04 test-02
```

Now execute `sudo lxc list` and you will get something similar to this:

Notice that both containers are located in different host? By default LXD will provision our containers on round-robin fashion. Depending on your requirements, this can be a major drawback. For some people it might be more beneficial if there's option to do load-based scheduling, but I understand that it is not supported as of now and also more complicated to setup because we have to gather metrics for that.

## Limitation and Conclusion

1. Can only schedule our container in round robin fashion
2. Operation data is not properly replicated across cluster (this is a bug)
