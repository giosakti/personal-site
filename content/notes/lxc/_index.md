---
title: "LXC"
authors: ["gio"]
---

#### Installing latest LXD version using Snap

By default Ubuntu 16.04 ships with LXD 2.x. If we want to utilize clustering, it means that we have to install LXD 3.0. The easiest way of doing this is using snap:

```shell
sudo snap install lxd --channel=3.0/stable
sudo lxd.migrate
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
sudo snap switch lxd --channel=<channel>
sudo snap refresh
```

We can use `snap info lxd` command to see the list of all possible channels.

> NOTE: Once upgraded, it cannot be downgraded anymore. It may break installed LXD.

#### Container Special Requirements

**Allow containers to consume more memory map areas**

```shell
sudo sysctl -w vm.max_map_count=262144
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
sudo systemctl daemon-reload
sudo systemctl restart snap.lxd.daemon
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
sudo lxc storage show local --target <lxc-hostname>
```

**Error: no "source" property found for the storage pool**

If we encounter this error on LXD log, then we can check if `storage_pools_config` exist properly on the database by typing this command:

```shell
lxd sql global "SELECT * FROM storage_pools_config;"
```

If it indeed missing, then we can add missing configuration manually:

```shell
lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'source', 'local');"
lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'volatile.initial_source', '/dev/xvdf');"
lxd sql global "INSERT INTO storage_pools_config (storage_pool_id, node_id, key, value) VALUES (1, 3, 'zfs.pool_name', 'local');"
```

See [this](https://discuss.linuxcontainers.org/t/lxd-cluster-hangs/1520/9) issue for details.

#### LXD behavior on Host reboot

Yes, on shutdown of the host all containers are sent a shutdown signal to which they have 30s to respond by doing a clean shutdown of the container. After that, if the container is still running, it'll be killed by LXD.

On startup, all containers which were running at the time the system was shut down will be started back up.

#### LXC Networking

Fan network is managed by lxd directly, see [here](https://lxd.readthedocs.io/en/stable-3.0/networks/).

If we restart LXD, network will restarted.

#### Cannot delete LXD zfs backed containers: dataset is busy

```
# Get process that is still mounting the volume
grep containers/test1 /proc/*/mountinfo

# Get rid of the mount
nsenter -t <PID> -m -- umount /var/lib/lxd/storage-pools/lxd/containers/test1
```

See [this](https://github.com/lxc/lxd/issues/4656) issue for details.

#### LXD: Unexpected response type 6

If you receive similar message like this during execution of any `lxc` command.

```
STDERR: Error: Failed to setup cluster trust: failed client cert to cluster: failed to notify peer <IP ADDRESS>:8443: cannot fetch node config from database: unexpected response type 6
```

Then you may have to restart lxd in the node specified by the message, by executing:

```
sudo systemctl reload snap.lxd.daemon
```

References:

- https://github.com/lxc/lxd/issues/3158
- https://github.com/lxc/lxd/issues/4947
- https://github.com/lxc/lxd/issues/5187
