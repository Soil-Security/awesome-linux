# Linux Control Groups (Cgroups) Cheat Sheet

1. Determine which version(s) are running on a host by verifying the mounted filesystems.
   You will see one line for "cgroup2" and several for "cgroup," indicating that both systems are installed.

   ``` console
   $ mount | grep cgroup
   cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
   ```
   Alternatively, run the `stat` command:
   ```
   stat -fc %T /sys/fs/cgroup
   ```
   For cgroup v2, the output is `cgroup2fs`. For cgroup v1, the output is `tmpfs`.
2. View all of the v2 cgroups in your Linux system in a tree rendering:

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

3. Recursively show control group contents:

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

4. Show top control groups by their resource usage:

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

5. Find cgroup v2 directory for a Kubernets pod:
   1. Find the container ID:
      ``` console
      $ sudo crictl ps
      CONTAINER      IMAGE          CREATED        STATE    NAME      ATTEMPT  POD ID         POD
      305a095f0ad44  da87ed6410cb8  7 minutes ago  Running  workload  2        5c8fcd9881d1a  petclinic-965647cb6-px9t
      ```
   2. Find systemd slice and scope:
      ``` console
      $ systemd-cgls --unit kubepods.slice --no-pager
      Unit kubepods.slice (/kubepods.slice):
      └─kubepods-burstable.slice 
        ├─kubepods-besteffort-pod51528708_82b0_4669_adc8_48b4dec6b4da.slice 
        │ ├─cri-containerd-5c8fcd9881d1af91314c8f635def98f0472e456c76fe468dceebaae6df3f2937.scope 
        │ │ └─84661 /pause
        │ └─cri-containerd-305a095f0ad44565a386ef99ff58e379460b98988f243502dbc6b868819c6404.scope 
        │   └─85410 java org.springframework.boot.loader.launch.JarLauncher
        └─kubepods-besteffort-pod5a4ff03d_7a6f_44e0_a767_6ac7ae8c047d.slice 
          ├─cri-containerd-4a65218ad97a7f0e055cdf5156a9014ac9fde12834b10042adfa1ecdd9cbe750.scope 
          │ ├─84711 postgres
          │ ├─84926 postgres: checkpointer
          │ ├─84927 postgres: background writer
          │ ├─84929 postgres: walwriter
          │ ├─84930 postgres: autovacuum launcher
          │ ├─84931 postgres: logical replication launcher
          │ ├─85540 postgres: user petclinic 192.168.47.215(59386) idle
          │ ├─85544 postgres: user petclinic 192.168.47.215(59400) idle
          │ ├─85545 postgres: user petclinic 192.168.47.215(59408) idle
          │ ├─85546 postgres: user petclinic 192.168.47.215(59424) idle
          │ ├─85547 postgres: user petclinic 192.168.47.215(59438) idle
          │ ├─85548 postgres: user petclinic 192.168.47.215(59454) idle
          │ ├─85549 postgres: user petclinic 192.168.47.215(59464) idle
          │ ├─85550 postgres: user petclinic 192.168.47.215(59470) idle
          │ ├─85551 postgres: user petclinic 192.168.47.215(59486) idle
          │ └─85552 postgres: user petclinic 192.168.47.215(59498) idle
          └─cri-containerd-ad20138ddf9f11a1fe994f3adc9a504a6f09a98614f2c4bcc630754607bd1c22.scope 
            └─84644 /pause
      ```
   3. Show systemd unit for the container:
      ``` console
      $ systemctl show --no-pager cri-containerd-305a095f0ad44565a386ef99ff58e379460b98988f243502dbc6b868819c6404.scope
      TimeoutStopUSec=1min 30s
      Result=success
      RuntimeMaxUSec=infinity
      Slice=kubepods-besteffort-pod51528708_82b0_4669_adc8_48b4dec6b4da.slice
      ControlGroup=/kubepods.slice/kubepods-besteffort.slice/kubepods-besteffort-pod51528708_82b0_4669_adc8_48b4dec6b4da.slice/cri-containerd-305a095f0ad44565a386ef99ff58e379460b98988f243502dbc6b868819c6404.scope
      MemoryCurrent=469774336
      MemoryAvailable=13979320320
      CPUUsageNSec=25048781000
      ```
