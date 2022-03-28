---
title: "OSX"
authors: ["gio"]
---

#### Finding the cause of a battery drain

If you experiencing battery drain issue when the lid is closed. We can see the logs by executing this command:

```
pmset -g log|grep -e " Sleep " -e " Wake "
```

One of the possible cause for recent OSX is: "Find My Mac" operates even when the lid is closed. It will try to connect to known wi-fi and then send its position accordingly. It can be disable by executing this command:

```
sudo pmset -b tcpkeepalive 0
```

References:

- https://forums.macrumors.com/threads/battery-dies-in-sleep-since-macos-upgrade.2115680/
- https://apple.stackexchange.com/questions/334202/macbook-pro-2018-switch-from-sleep-to-darkwake-in-loop-how-to-diagnose
