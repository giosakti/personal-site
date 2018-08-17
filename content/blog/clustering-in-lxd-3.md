---
title: "Clustering in LXD 3.0"
authors: ["gio"]
date: 2018-08-16T20:00:00+07:00
tags:
- container
- LXD
- LXC
---

<em>I've been experimenting and working on a project using LXD in the past few months and there is an interesting feature that I want to talk about, which is the **clustering** ability that I found in LXD 3.0.</em>

When I started working on my project, I initially used LXD 2.x because that is what's available by default on Ubuntu 16.04. However, it turns out that my project needs multiple nodes to work together so I was looking at the options. Happily, LXD 3.0 which released on early April has support for clustering and I can now join my nodes together. It is also an LTS release, so it means that it will be supported until June 2023.

## Upgrading

If you're like me and still using Ubuntu 16.04, LXD 3.0 won't be available by default. One of the easiest option to upgrade (that I also use) is to use the snap package manager. LXD 3.0 package also comes with migration tool, which is really useful. But if you're doing the migration on production environment, **always be prepared for unexpected things**.

Install by executing the `snap install` command.

```shell
sudo snap install lxd --channel=3.0/stable
```

The command will install LXD on different location than the default LXD 2.x on Ubuntu 16.04. I pick `3.0/stable` channel because it is the LTS release, while any other 3.x channels are stable release and will be unsupported whenever new minor version comes out.

From now on, make sure that we are executing on the 3.0 version by typing `lxd -v`. And now we can call the migration command. 

The migrator is quite nice and seamless, it will also ask if we want to remove the old LXD version.

```shell
 sudo lxd.migrate
   Do you want to uninstall the old LXD (yes/no) [default=no]? yes
```

Because we want to try out clustering mode, we have to provision another node which also has its LXD upgraded to 3.0. 

> In production, you may want to cluster odd number of nodes with minimum number of 3 to be safe and to avoid deadlock in election (dqlite, database that LXD currently use for clustering uses [Raft](https://raft.github.io/)).

## Bootstrap the first node

Procedure for bootstrapping the first node will be slightly different than all the other node. There are two ways to bootstrap, first is to use the interactive 'wizard' by executing `sudo lxd init` and the other way is to use a 'preseed' file which will be used to setup. We will do the first way, so that we can understand the process more clearly in step-by-step fashion.

```shell
sudo lxd init
```

Now answer these questions, the first few questions should be obvious.

```shell
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What name should be used to identify this node in the cluster? [default=ubuntu-xenial]: node-01
What IP address or DNS name should be used to reach this node? [default=a.b.c.d]: <enter>
Are you joining an existing cluster? (yes/no) [default=no]: no
```

Next, we will be asked for cluster password. All the other node will use this password to register for the first time into this first node and to get the certificate. Put password in a safe place.

```shell
Setup password authentication on the cluster? (yes/no) [default=yes]: yes
Trust password for new clients: <type-your-password>
Again: <type-your-password>
```

We may define our storage pool now, we can set both local and remote storage pool. We can pick 4 different storage backends for our storage pool.

```shell
Do you want to configure a new local storage pool? (yes/no) [default=yes]: yes
Name of the storage backend to use (btrfs, dir, lvm, zfs) [default=zfs]: <pick-any>
Do you want to configure a new remote storage pool? (yes/no) [default=no]: no
```

MaaS is abstraction for physical servers, right now I don't use it

```shell
Would you like to connect to a MAAS server? (yes/no) [default=no]: no
```

Now we configure networking, we can either make LXD use existing bridge or let LXD create a new one. I already prepared a fan network beforehand, but if we put `no` here, we will be asked if we want to create a new fan network.

```shell
Would you like to configure LXD to use an existing bridge or host interface? (yes/no) [default=no]: yes
Name of the existing bridge or host interface: fan-10
```

Last 2 questions are self-explanatory.

```shell
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] no
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: yes
```

Congratulations, we finished the first node setup.

## Bootstrap the joining node

On joining node, we can bootstrap the same way by executing `sudo lxd init`, but the answer that we give should be slightly different. Let's try doing it and see what the wizard progression looks like. Make sure that you make note of the first node IP and cluster trust password before continuing.

```shell
sudo lxd init
```

The first few questions are familiar.

```shell
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What name should be used to identify this node in the cluster? [default=ubuntu-xenial]: node-2
What IP address or DNS name should be used to reach this node? [default=a.b.c.d]: <enter>
```

Now, we will be asked whether we want to join existing cluster. Put appropriate information here, including IP of the first node and the cluster trust password that we already setup on the first node before.

```shell
Are you joining an existing cluster? (yes/no) [default=no]: yes
IP address or FQDN of an existing cluster node: <ip-of-first-node>
Cluster fingerprint: xyz
You can validate this fingerpring by running "lxc info" locally on an existing node.
Is this the correct fingerprint? (yes/no) [default=no]: yes
Cluster trust password: <password>
All existing data is lost when joining a cluster, continue? (yes/no) [default=no] yes
```

Lastly, we will be asked for path to be used as local storage pool. If we're joining existing cluster, most other configuration will be copied from the first node, except this.

```shell
Choose the local disk or dataset for storage pool "local" (empty for loop disk): <enter>
```

## Testing

So now we have 2 node in clustering mode, great! We can try spinning up some containers and see where the containers will end up in.

```shell
sudo lxc launch ubuntu:16.04 test-01
sudo lxc launch ubuntu:16.04 test-02
```

Now execute `sudo lxc list` and you will get something similar to this:

```shell
+---------+---------+--------------------+-+------------+-----------+------------+
|  NAME   |  STATE  |        IPV4        | |    TYPE    | SNAPSHOTS |  LOCATION  |
+---------+---------+--------------------+-+------------+-----------+------------+
| test-01 | RUNNING | aa.bb.cc.xx (eth0) | | PERSISTENT |     0     | node-1     |
+---------+---------+--------------------+-+------------+-----------+------------+
| test-02 | RUNNING | aa.bb.cc.yy (eth0) | | PERSISTENT |     0     | node-2     |
+---------+---------+--------------------+-+------------+-----------+------------+
```

Notice that both containers are located in different node? By default LXD will provision our containers in **round-robin fashion**. Depending on your requirements, this can be a major drawback. For some people it might be more beneficial if there's option to do load-based scheduling, but I understand that it is not supported as of now and also more complicated to setup because we have to gather metrics first. 

*On a side, I'm doing something interesting related to that, perhaps I will discuss about what I'm working on in my future post.*

## Limitation and Conclusion

There are couple of things that limits the ability of current version of LXD clustering. First is that it can only schedule our container in round robin fashion. We can target a specific node everytime we want to launch a container, but then we have to roll out our own scheduler.

The other thing is I found a bug when working with the LXD API. So every operation in LXD is **done asynchronously**, including container creation and start operation. When we do any operation, the API will return the operation ID and we have to query periodically to determine if our operation is a success. The problem is, the operation itself will only exist in node where the operation is performed. In other node, the operation won't be replicated and we will get a 404. This has been recognized as a bug, which you can find more info about it [here](https://github.com/lxc/lxd/issues/4721). Fixes are rolled out in LXD 3.3, unfortunately that is a stable channel not an LTS channel. So now I have to weight decisions whether to upgrade or not. Because we cannot rollback once we upgrade.

All in all, LXD is heading into a nice direction with very active development team. They do stable release almost every month and many major features and fixes were introduced in every stable version. I'm seeing a potential with this platform and thus will be using it for a while.

## References

- https://blog.ubuntu.com/2018/05/03/lxd-clusters-a-primer
- https://lxd.readthedocs.io/en/latest/clustering/
- https://discuss.linuxcontainers.org/t/lxd-3-0-0-has-been-released/1491
- https://stgraber.org/category/lxd/
