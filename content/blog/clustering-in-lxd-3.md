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

The command will install LXD on a different location than the default LXD on Ubuntu 16.04.

Make sure that we are executing on the 3.0 version by typing `lxd -v`. And now we can do the migration, the migration process is nice because LXD will automatically migrate our existing containers and it will also ask if we want to remove the old LXD version.

```shell
```

Because we want to try out the clustering mode, we have to provision another machine which also has its LXD upgraded to 3.0. 

> In production, you may want to cluster odd number of machines with minimum number of 3 to be safe and to avoid deadlock in election (dqlite, database that LXD currently use for clustering uses Raft).

## Bootstrap the first node

Procedure for bootstrapping the first node will be slightly different than all the other node. There are two ways to bootstrap, first is to use the interactive 'wizard' by executing `sudo lxd init` and the other way is to use a 'preseed' file which will be used to setup. We will do the first way, so that we can understand the process more clearly in step-by-step fashion.

```shell
sudo lxd init
```

## Bootstrap the joining node

On joining node, we can bootstrap the same way by executing `sudo lxd init`, but the answer that we give should be slightly different. Let's try doing it and see what the wizard progression looks like. Make sure that you make note of the first node IP and cluster trust password before continuing.

```shell
sudo lxd init
```

## Testing

So now we have 2 host in clustering mode, great! We can try spinning up some containers and see where the containers will end up in.

```shell
sudo lxc launch ubuntu:16.04 test-01
sudo lxc launch ubuntu:16.04 test-02
```

Now execute `sudo lxc list` and you will get something similar to this:

Notice that both containers are located in different host? By default LXD will provision our containers in round-robin fashion. Depending on your requirements, this can be a major drawback. For some people it might be more beneficial if there's option to do load-based scheduling, but I understand that it is not supported as of now and also more complicated to setup because we have to gather metrics first. 

On a side, I'm doing something interesting related to that, perhaps I will discuss about what I'm working on in my future post.

## Limitation and Conclusion

There are couple of things that limits the ability of current version of LXD clustering. First is it can only schedule our container in round robin fashion. We can target a specific host everytime we want to launch a container, but then we have to roll out our own scheduler.

The other thing is I found a bug when working with the LXD API. So every operation in LXD is done asynchronously, including container creation and start operation. When we do any operation, the API will return the operation ID and we have to manually query periodically to determine if our operation is a success. The problem is, the operation itself will only exist in machine where the operation is performed. In other machine, the operation won't be replicated and we will get a 404. This has been recognized as a bug, which you can find more info about it [here](). Fixes are rolled out in LXD 3.3, unfortunately that is a stable channel not an LTS channel. So now I have to weight decisions whether to upgrade or not. Because we cannot rollback once we upgrade.

All in all, LXD is heading into a nice direction with very active development team. They do stable release almost every month and many major features and fixes were introduced in every stable version. I'm seeing a potential with this platform, which may fill some niches if we compared it to docker.

## References
