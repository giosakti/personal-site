---
title: "ZFS"
authors: ["gio"]
---

#### Resize disk online

1. Resize the physical disk
2. Do these commands within the node itself

```
sudo partprobe -s
sudo partprobe
lsblk
zpool list
sudo zpool online -e <POOL NAME> <DEVICE NAME>
zpool list

# Note:
# partprobe - command to inform the OS of partition table changes
```

Reference: https://serverfault.com/questions/798907/growing-zpool-in-zfsonlinux
