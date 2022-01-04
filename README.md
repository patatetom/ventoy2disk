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
fdisk -W always $TARGET <<~~~~~
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

```console
mkfs.exfat -n Ventoy ${TARGET}p1
```
