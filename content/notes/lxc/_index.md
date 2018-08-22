---
title: "LXC"
authors: ["gio"]
---

#### Installing latest LXD version using Snap

By default Ubuntu 16.04 ships with LXD 2.x. If we want to utilize clustering, it means that we have to install LXD 3.0. The easiest way of doing this is using snap:

```shell
$ sudo snap install lxd --channel=3.0/stable
$ sudo lxd.migrate
  Do you want to uninstall the old LXD (yes/no) [default=no]? yes
```

Log out then log back in, after that do `sudo lxd init` and answer the questions.

We can also create a configuration file then initialize lxd with it using this command:

```shell
cat /etc/default/lxd_preseed.yml | sudo lxd init --preseed
```

#### Upgrading LXD

If we want to upgrade, we can track another channel and do a refresh by doing this command:

```shell
$ sudo snap switch lxd --channel=<channel>
$ sudo snap refresh
```

We can use `snap info lxd` command to see the list of all possible channels.

> NOTE: Once upgraded, it cannot be downgraded anymore. It may break installed LXD.

#### Container Special Requirements

**Allow containers to consume more memory map areas**

```shell
$ sudo sysctl -w vm.max_map_count=262144
```

To make it persist you can also do:

```shell
# Ensure this line exist at the bottom of /etc/sysctl.conf

...
vm.max_map_count=262144
```

Reboot. To verify after rebooting, type `sysctl vm.max_map_count`.

**Allow containers to lock unlimited memory**

```shell
# Ensure this line exist in '/etc/systemd/system/snap.lxd.daemon.service'

[Service]
...
LimitMEMLOCK=infinity
```

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart snap.lxd.daemon
```

#### LXD Verbose Mode

If we want to see every outputs that LXD made, we can startup in verbose mode. Do this:

1. Stop the daemon

    ```shell
    systemctl stop snap.lxd.daemon

    # or force stop

    sudo service snap.lxd.daemon force-stop

    # or kill it as necessary
    ```

2. Start the daemon in verbose mode

    ```shell
    LXD_DIR=/var/snap/lxd/common/lxd LD_LIBRARY_PATH=/snap/lxd/current/lib/ /snap/lxd/current/bin/lxd --verbose --debug
    ```

#### Monitor LXC While Running

```shell
lxc monitor --pretty --type logging
```

#### Database

LXD uses special, distributed version of sqlite called [dqlite](https://github.com/CanonicalLtd/dqlite). LXD has local and global database running on clustering mode.

We can do a manual query to this database on special circumstances. Example:

```shell
echo "UPDATE config SET value='aa.bb.cc.dd:8443' WHERE key='core.https_address'" | sqlite3 /var/snap/lxd/common/lxd/database/local.db
```

#### Storage Pools

**See list of local volumes on an LXC host**

```shell
$ sudo lxc storage show local --target <lxc-hostname>
```

**Error: no "source" property found for the storage pool**

If we encounter this error on LXD log, then we can check if `storage_pools_config` exist properly on the database by typing this command:

```shell
$ lxd sql global "SELECT * FROM storage_pools_config;"
```

If it indeed missing, then we can add missing configuration manually:

```shell
$ lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'source', 'local');"
$ lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'volatile.initial_source', '/dev/xvdf');"
$ lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'zfs.pool_name', 'local');"
```

See [this](https://discuss.linuxcontainers.org/t/lxd-cluster-hangs/1520/9) issue for details.
