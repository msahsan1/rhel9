<pre>

<h1>  Creating Stratis Volumes </h1>

[root@vm-rhel9 rhel9]# dnf install stratisd stratis-cli
Updating Subscription Management repositories.
Last metadata expiration check: 0:36:13 ago on Wed 07 Feb 2024 09:40:36 AM.
Dependencies resolved.
==================================================================================================================================================
 Package                                       Architecture        Version                    Repository                                     Size
==================================================================================================================================================
Installing:
 stratis-cli                                   noarch              3.5.3-1.el9                rhel-9-for-x86_64-appstream-rpms              130 k
 stratisd                                      x86_64              3.5.8-1.el9                rhel-9-for-x86_64-appstream-rpms              4.4 M
Installing dependencies:
 python3-dbus-client-gen                       noarch              0.5.1-1.el9                rhel-9-for-x86_64-appstream-rpms               31 k
 python3-dbus-python-client-gen                noarch              0.8.3-1.el9                rhel-9-for-x86_64-appstream-rpms               33 k
 python3-dbus-signature-pyparsing              noarch              0.4.1-1.el9                rhel-9-for-x86_64-appstream-rpms               24 k
 python3-into-dbus-python                      noarch              0.8.2-1.el9                rhel-9-for-x86_64-appstream-rpms               34 k
 python3-justbases                             noarch              0.15.2-1.el9               rhel-9-for-x86_64-appstream-rpms               53 k
 python3-justbytes                             noarch              0.15.2-1.el9               rhel-9-for-x86_64-appstream-rpms               49 k
 python3-packaging                             noarch              20.9-5.el9                 rhel-9-for-x86_64-appstream-rpms               81 k
 python3-pyparsing                             noarch              2.4.7-9.el9                rhel-9-for-x86_64-baseos-rpms                 154 k
 python3-wcwidth                               noarch              0.2.5-8.el9                rhel-9-for-x86_64-appstream-rpms               48 k

Transaction Summary
==================================================================================================================================================
Install  11 Packages

Total download size: 5.0 M
Installed size: 21 M
Is this ok [y/N]:

 systemctl enable --now stratisd

[root@vm-rhel9 rhel9]#  stratis pool list
Name   Total / Used / Free   Properties   UUID   Alerts
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0  108G  0 disk
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0  107G  0 part
  ├─rhel-root 253:0    0 69.2G  0 lvm  /
  ├─rhel-swap 253:1    0  3.9G  0 lvm  [SWAP]
  └─rhel-home 253:2    0 33.8G  0 lvm  /home
sdb             8:16   0   20G  0 disk
sdc             8:32   0   20G  0 disk
sr0            11:0    1 1024M  0 rom
[root@vm-rhel9 rhel9]#  stratis pool create mypool /dev/sdb
[root@vm-rhel9 rhel9]#  stratis pool list
Name                 Total / Used / Free    Properties                                   UUID   Alerts
mypool   20 GiB / 532.44 MiB / 19.48 GiB   ~Ca,~Cr, Op   83b3c646-98d8-43c8-b31f-aa166f399b1b
[root@vm-rhel9 rhel9]#

<h2> Extend Pool </h2>

[root@vm-rhel9 rhel9]#  stratis pool add-data mypool /dev/sdc
[root@vm-rhel9 rhel9]#  stratis pool
Name                 Total / Used / Free    Properties                                   UUID   Alerts
mypool   40 GiB / 546.62 MiB / 39.47 GiB   ~Ca,~Cr, Op   83b3c646-98d8-43c8-b31f-aa166f399b1b
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]#  stratis blockdev list
Pool Name   Device Node   Physical Size   Tier   UUID
mypool      /dev/sdb             20 GiB   DATA   b9a951d0-65e2-41e0-943b-611c5de7494f
mypool      /dev/sdc             20 GiB   DATA   170fd962-e67d-47a9-9fc6-429988e30fc1
[root@vm-rhel9 rhel9]#

<h2> Create FileSyetm </h2>

[root@vm-rhel9 rhel9]#  stratis fs create mypool myfs
[root@vm-rhel9 rhel9]# mkdir /myfs
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]#  stratis fs list
Pool     Filesystem   Total / Used / Free             Created             Device                     UUID
mypool   myfs         1 TiB / 546 MiB / 1023.47 GiB   Feb 07 2024 10:30   /dev/stratis/mypool/myfs   e6ad0a31-3d5f-47ff-b437-bc481e2341ec
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed Feb  7 17:23:34 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=bd6cc4cb-b111-48cf-bb47-0be23b1638b2 /boot                   xfs     defaults        0 0
/dev/mapper/rhel-home   /home                   xfs     defaults        0 0
/dev/mapper/rhel-swap   none                    swap    defaults        0 0
UUID=e6ad0a31-3d5f-47ff-b437-bc481e2341ec        /myfs  xfs     x-systemd.requires=stratisd.service 0 0
[root@vm-rhel9 rhel9]# mount -a
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]# mount -a
[root@vm-rhel9 rhel9]# df -h
Filesystem                                                                                       Size  Used Avail Use% Mounted on
devtmpfs                                                                                         4.0M     0  4.0M   0% /dev
tmpfs                                                                                            1.8G     0  1.8G   0% /dev/shm
tmpfs                                                                                            731M  8.7M  723M   2% /run
/dev/mapper/rhel-root                                                                             70G  2.2G   68G   4% /
/dev/mapper/rhel-home                                                                             34G  274M   34G   1% /home
/dev/sda1                                                                                       1014M  244M  771M  25% /boot
tmpfs                                                                                            366M     0  366M   0% /run/user/0
tmpfs                                                                                            366M     0  366M   0% /run/user/1000
tmpfs                                                                                            1.0M     0  1.0M   0% /run/stratisd/ns_mounts
/dev/mapper/stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-e6ad0a313d5f47ffb437bc481e2341ec  1.0T  7.2G 1017G   1% /myfs
[root@vm-rhel9 rhel9

<h1> Stratis Snapshots </h1>

[root@vm-rhel9 rhel9]# dd if=/dev/zero of=/myfs/bigfile bs=1M count=2000
2000+0 records in
2000+0 records out
2097152000 bytes (2.1 GB, 2.0 GiB) copied, 3.6107 s, 581 MB/s
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]#  stratis pool list
Name               Total / Used / Free    Properties                                   UUID   Alerts
mypool   40 GiB / 3.02 GiB / 36.98 GiB   ~Ca,~Cr, Op   83b3c646-98d8-43c8-b31f-aa166f399b1b

[root@vm-rhel9 rhel9]#  stratis pool list
Name               Total / Used / Free    Properties                                   UUID   Alerts
mypool   40 GiB / 3.02 GiB / 36.98 GiB   ~Ca,~Cr, Op   83b3c646-98d8-43c8-b31f-aa166f399b1b
[root@vm-rhel9 rhel9]#  stratis fs list
Pool     Filesystem   Total / Used / Free              Created             Device                     UUID
mypool   myfs         1 TiB / 2.49 GiB / 1021.51 GiB   Feb 07 2024 10:30   /dev/stratis/mypool/myfs   e6ad0a31-3d5f-47ff-b437-bc481e2341ec
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]#  stratis fs snapshot mypool myfs myfs-snap
[root@vm-rhel9 rhel9]#  stratis fs list
Pool     Filesystem   Total / Used / Free              Created             Device                          UUID
mypool   myfs         1 TiB / 2.49 GiB / 1021.51 GiB   Feb 07 2024 10:30   /dev/stratis/mypool/myfs        e6ad0a31-3d5f-47ff-b437-bc481e2341ec
mypool   myfs-snap    1 TiB / 2.49 GiB / 1021.51 GiB   Feb 07 2024 10:43   /dev/stratis/mypool/myfs-snap   f32ec272-46d4-44c4-8835-2289e5201b02
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]# rm /myfs/bigfile
rm: remove regular file '/myfs/bigfile'? y
[root@vm-rhel9 rhel9]# mkdir /myfs-snap
[root@vm-rhel9 rhel9]# mount /dev/stratis/mypool/myfs-snap /myfs-snap/
[root@vm-rhel9 rhel9]# ls -l /myfs-snap/
total 2048000
-rw-r--r--. 1 root root 2097152000 Feb  7 10:41 bigfile
[root@vm-rhel9 rhel9]#

[root@vm-rhel9 rhel9]# ls -l /myfs-snap/
total 2048000
-rw-r--r--. 1 root root 2097152000 Feb  7 10:41 bigfile
[root@vm-rhel9 rhel9]# df -h
Filesystem                                                                                       Size  Used Avail Use% Mounted on
devtmpfs                                                                                         4.0M     0  4.0M   0% /dev
tmpfs                                                                                            1.8G     0  1.8G   0% /dev/shm
tmpfs                                                                                            731M  8.7M  723M   2% /run
/dev/mapper/rhel-root                                                                             70G  2.2G   68G   4% /
/dev/mapper/rhel-home                                                                             34G  274M   34G   1% /home
/dev/sda1                                                                                       1014M  244M  771M  25% /boot
tmpfs                                                                                            366M     0  366M   0% /run/user/0
tmpfs                                                                                            366M     0  366M   0% /run/user/1000
tmpfs                                                                                            1.0M     0  1.0M   0% /run/stratisd/ns_mounts
/dev/mapper/stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-e6ad0a313d5f47ffb437bc481e2341ec  1.0T  7.2G 1017G   1% /myfs
/dev/mapper/stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-f32ec27246d444c488352289e5201b02  1.0T  9.2G 1015G   1% /myfs-snap
[root@vm-rhel9 rhel9]#

<h2> Destroy Pool </h2>

[root@vm-rhel9 ~]#  stratis fs destroy mypool myfs-snap
Execution failed:
stratisd failed to perform the operation that you requested. It returned the following information via the D-Bus: ERROR: low-level ioctl error due to nix error; ioctl number: 4, input header: Some(DeviceInfo { version: Version { major: 4, minor: 0, patch: 0 }, data_size: 16384, data_start: 312, target_count: 0, open_count: 0, flags: (empty), event_nr: 4253434, dev: Device { major: 0, minor: 0 }, name: Some(DmNameBuf { inner: "stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-f32ec27246d444c488352289e5201b02" }), uuid: None }), header result: Some(DeviceInfo { version: Version { major: 4, minor: 46, patch: 0 }, data_size: 16384, data_start: 312, target_count: 0, open_count: 0, flags: (empty), event_nr: 4253434, dev: Device { major: 0, minor: 0 }, name: Some(DmNameBuf { inner: "stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-f32ec27246d444c488352289e5201b02" }), uuid: None }), error: EBUSY: Device or resource busy.

[root@vm-rhel9 ~]# cd
[root@vm-rhel9 ~]# umount /myfs-snap
[root@vm-rhel9 ~]#  stratis fs destroy mypool myfs-snap
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# df -h
Filesystem                                                                                       Size  Used Avail Use% Mounted on
devtmpfs                                                                                         4.0M     0  4.0M   0% /dev
tmpfs                                                                                            1.8G     0  1.8G   0% /dev/shm
tmpfs                                                                                            731M  8.7M  723M   2% /run
/dev/mapper/rhel-root                                                                             70G  2.2G   68G   4% /
/dev/mapper/rhel-home                                                                             34G  274M   34G   1% /home
/dev/sda1                                                                                       1014M  244M  771M  25% /boot
tmpfs                                                                                            366M     0  366M   0% /run/user/0
tmpfs                                                                                            366M     0  366M   0% /run/user/1000
tmpfs                                                                                            1.0M     0  1.0M   0% /run/stratisd/ns_mounts
/dev/mapper/stratis-1-83b3c64698d843c8b31faa166f399b1b-thin-fs-e6ad0a313d5f47ffb437bc481e2341ec  1.0T  7.2G 1017G   1% /myfs
[root@vm-rhel9 ~]#


</pre>
