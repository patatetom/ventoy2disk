# ventoy2disk

_ventoy to disk revisited (https://github.com/ventoy/Ventoy)_

manual installation of Ventoy in BIOS mode. This mode remains perfectly functional on a UEFI machine, which is not necessarily the case with UEFI mode on a BIOS machine.


## prerequisites

- `boot.img` (Ventoy)
- `core.img.xz` (Ventoy)
- `VTOYEFI.part.img.xz`
- `bash` with root access
- `dd`
- `exfat-utils` (Debian)
- `fdisk`
- `grep` (installation script)
- `umount`
- `util-linux` (Debian)
- `xz-utils` (Debian)

> the image of the partition `VTOYEFI.part.img.xz` is derived from that of `ventoy.disk.img.xz`.


## partitioning

```console
TARGET=/dev/todo
umount $TARGET*
BLOCKS=$( blockdev --getsz $TARGET )
fdisk $TARGET <<~~~~~
o
n
p
1
2048
$(( BLOCKS - 65537 ))
t
7
n
p
2
$(( BLOCKS - 65536 ))
$(( BLOCKS - 1 ))
t
2
ef
a
1
w
~~~~~
```


## installation of Ventoy components

```console
dd if=boot.img of=$TARGET bs=1 count=446
xzcat core.img.xz | dd of=$TARGET count=2047 seek=1
xzcat VTOYEFI.part.img.xz | dd of=$TARGET count=65536 seek=$(( BLOCKS - 65536 ))
```


## random serial number

> this step is optional.

```console
dd if=/dev/urandom of=$TARGET seek=384 bs=1 count=16
dd if=/dev/urandom of=$TARGET seek=440 bs=1 count=4
```


## ExFAT formatting of the first partition

> depending on the device, the first partition may not named `p1` but simply `1` (for example `/dev/sde1` with `/dev/sde`).

```console
lsblk $TARGET
# adapt the next command with the results of the previous command (see note above)
mkfs.exfat -n Ventoy ${TARGET}p1
```



---



# mount Ventoy storage partition
_from a Linux live system started from Ventoy._

```console
$ # switch user (eg. root)
$ sudo -i

# # list Ventoy USB device
# lsblk -o+fstype /dev/sda
NAME       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS       FSTYPE
sda          8:0    1 57,8G  0 disk                   
├─sda1       8:1    1 57,7G  0 part                   exfat
│ └─ventoy 254:0    0  3,7G  1 dm   /run/miso/bootmnt iso9660
└─sda2       8:2    1   32M  0 part                   vfat

# # create mount point
# mkdir /storage

# try to mount Ventoy storage partition
# mount /dev/sda1 /storage
mount: /storage: fsconfig system call failed: /dev/sda1: Can't open blockdev.
       dmesg(1) may have more information after failed mount system call.

# # can't mount Ventoy storage partition because already used
# lsblk -Q "pkname=='sda1'"
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
ventoy 254:0    0  3,7G  1 dm   /run/miso/bootmnt

# # insert "magic" nbd kernel module
# modprobe nbd

# # install qemu-nbd if not provided by the used Linux live distribution
# # (apt install qemu-utils # for example on Debian)
# # connect sda with nbd0
# qemu-nbd -c /dev/nbd0 /dev/sda
WARNING: Image format was not specified for '/dev/sda' and probing guessed raw.

# # mount nbd0p1 (eg. sda1) with uid 1000 (default live user)
# mount /dev/nbd0p1 /storage -o uid=1000
# exit


$ # play with your Ventoy storage partition (eg. /storage) :
$ # save your downloads,
$ # edit your files,
$ # etc..


$ # unmount everything
$ sudo -i
# umount /storage
# qemu-nbd -d /dev/nbd0
# exit
```
