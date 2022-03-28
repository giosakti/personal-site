---
title: "Linux"
authors: ["gio"]
---

#### Check Memory Usage Info from File

```
cat /proc/meminfo

# Filter information
egrep --color 'Mem|Cache|Swap' /proc/meminfo
```

References:

- https://www.cyberciti.biz/faq/linux-check-memory-usage/

#### Configure User Space Hibernation

**Allocate swapfile**

```
sudo fallocate -l 24G /swapfile
sudo mkswap /swapfile

# add to /etc/fstab
/swapfile  none  swap  sw 0  0

# Set swappiness
sudo sysctl -w vm.swappiness=1

# Turn on swapfile
sudo swapon /swapfile
```

**Install and Configure User-space Software Suspend**

```
sudo apt install uswsusp
sudo dpkg-reconfigure -pmedium uswsusp

# Answer these when asked
Continue without a valid swap space? yes
Swap space to resume from: select partition where the swapfile is created
Encrypt? no

# Check generated /etc/uswsusp.conf
# resume offset is where the swapfile actually is
# to check the offset, run:
sudo swap-offset /swapfile

# After editing /etc/uswsusp.conf, run:
sudo update-initramfs -u

# Test hibernation
sudo s2disk
```

**Allow Resume to Happen** 

```
# Edit /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="resume=UUID=<PARTITION-UUID> resume_offset=<SWAPFILE-OFFSET> quiet"

# For Pop OS! only
sudo kernelstub -a "resume=UUID=<PARTITION-UUID> resume_offset=<SWAPFILE-OFFSET>" -d "splash"
```

**References**

- https://www.reddit.com/r/pop_os/comments/alqkfl/any_news_on_hibernate_feature_for_laptops_w_pop_os/
- https://wiki.debian.org/Hibernation/Hibernate_Without_Swap_Partition
