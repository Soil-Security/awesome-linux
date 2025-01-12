# Linux Control Groups (Cgroups) Cheat Sheet

Determine which version(s) are running on a host by verifying the mounted filesystems.
You will see one line for "cgroup2" and several for "cgroup," indicating that both systems are installed.

``` console
$ mount | grep cgroup
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
```

View all of the v2 cgroups in your Linux system in a tree rendering:

``` console
$ systemctl status
● miggodor
    State: running
    Units: 661 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Sun 2025-01-12 20:48:35 CET; 33min ago
  systemd: 255.4-1ubuntu8.4
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init splash
           ├─system.slice
           │ ├─ModemManager.service
           │ │ └─1660 /usr/sbin/ModemManager
           │ ├─NetworkManager.service
           │ │ └─1606 /usr/sbin/NetworkManager --no-daemon
           │ ├─accounts-daemon.service
           │ │ └─1567 /usr/libexec/accounts-daemon
           │ ├─avahi-daemon.service
           │ │ ├─1546 "avahi-daemon: running [miggodor.local]"
           │ │ └─1580 "avahi-daemon: chroot helper"
           │ ├─colord.service
[...]
```

Recursively show control group contents:

``` console
$ systemd-cgls
CGroup /:
-.slice
├─user.slice
│ └─user-1000.slice
│   ├─user@1000.service …
│   │ ├─session.slice
│   │ │ ├─gvfs-goa-volume-monitor.service
│   │ │ │ └─3478 /usr/libexec/gvfs-goa-volume-monitor
│   │ │ ├─org.gnome.SettingsDaemon.MediaKeys.service
│   │ │ │ └─3171 /usr/libexec/gsd-media-keys
│   │ │ ├─org.gnome.SettingsDaemon.Smartcard.service
│   │ │ │ └─3185 /usr/libexec/gsd-smartcard
│   │ │ ├─xdg-permission-store.service
│   │ │ │ └─2863 /usr/libexec/xdg-permission-store
│   │ │ ├─org.gnome.SettingsDaemon.Datetime.service
│   │ │ │ └─3165 /usr/libexec/gsd-datetime
│   │ │ ├─filter-chain.service
│   │ │ │ └─2802 /usr/bin/pipewire -c filter-chain.conf
│   │ │ ├─xdg-document-portal.service
│   │ │ │ ├─2859 /usr/libexec/xdg-document-portal
│   │ │ │ └─2870 fusermount3 -o rw,nosuid,nodev,fsname=portal,auto_unmount,subtype=portal -- /run/user/1000/doc
│   │ │ ├─org.gnome.SettingsDaemon.Housekeeping.service
│   │ │ │ └─3166 /usr/libexec/gsd-housekeeping
│   │ │ ├─xdg-desktop-portal.service
[...]
```

Show top control groups by their resource usage:

``` console
$ systemd-cgtop
CGroup                                            Tasks   %CPU   Memory  Input/s Output/s
user.slice                                          967    7.4     4.0G        -        -
user.slice/user-1000.slice                          967    7.3     3.7G        -        -
user.slice/user-1000.slice/user@1000.service        955    7.3     3.7G        -        -
/                                                  1385    7.0     3.5G        -        -
system.slice                                        141    0.8   747.0M        -        -
system.slice/tailscaled.service                      15    0.5    45.9M        -        -
system.slice/containerd.service                      10    0.1    54.2M        -        -
system.slice/open-vm-tools.service                    4    0.1     7.4M        -        -
system.slice/systemd-oomd.service                     1    0.0     1.9M        -        -
system.slice/docker.service                          14    0.0   113.3M        -        -
dev-hugepages.mount                                   -      -   116.0K        -        -
dev-mqueue.mount                                      -      -     4.0K        -        -
init.scope                                            1      -    20.3M        -        -
proc-sys-fs-binfmt_misc.mount                         -      -     8.0K        -        -
sys-fs-fuse-connections.mount                         -      -     4.0K        -        -
sys-kernel-config.mount                               -      -     4.0K        -        -
[...]
```
