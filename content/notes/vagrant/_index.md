---
title: "Vagrant"
authors: ["gio"]
---

#### Optimize Synced Folders

The NFS options that impact the speed of the synced folder can be separated in two categories:

1. Mount options (guest side):
"udp" -> "tcp": The overhead incurred by TCP over UDP usually slows things down. However, it seems that the performance are slightly better with TCP in this particular case. (speed x1.5)

2. NFSd options (host side):
"sync" -> "async": in asynchronous mode, your host will acknowledge write requests before they are actually written onto disk. With a virtual machine, the network link between the host and guest is perfect so that there is no risk of data corruption. (speed x3)

If you want to override one option, you also need to write all the other default options. The optimal configuration in my situation is therefore:

```shell
config.vm.synced_folder "src/", "/srv/website", type: "nfs",
  mount_options: ['rw', 'vers=3', 'tcp'],
  linux__nfs_options: ['rw','no_subtree_check','all_squash','async']
```

References:

1. https://blog.theodo.fr/2017/07/speed-vagrant-synced-folders/
