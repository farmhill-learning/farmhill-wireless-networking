# Configuring the Disk

The router has very limited amount of disk space and it is not sufficient to install additional software. The recommended way to address this limitation is using storage device to expand the root file system.

It is a slightly tricky setup, documented in the [Extroot configuration][1] page of the openWrt documentation. It is done in a way that even if we remove the disk, the router can function with the mininal settings, without the additional software.

[1]: https://openwrt.org/docs/guide-user/additional-software/extroot_configuration

This page is adopted from the [Extroot confguration][1] page with a some changes to how the partitioning is done.

## Partitioning and Formatting the Disk

Using the Disk application in Ubuntu partition the disk.

Create an ext4 partition with the all the available space after leaving about one GB for swap. Label it as `extroot`.

Create a swap patition with the remaining space.

It is convenient to have the ext4 partition at the beginning. That will be make the device name `/dev/sda1`.

## Preparation

Install the required software to mount the ext4 file system.

Login to the router using the following command and use the password that you have set for the router in the web interface to login.

```
ssh root@192.168.1.1
```

After logging in, run the following commands:

```
opkg update
opkg install block-mount kmod-fs-ext4 e2fsprogs kmod-usb-storage
```

## Verifying

Plugin in the USB disk and run the command `block info`.

```
root@OpenWrt:~# block info
/dev/ubiblock0_0: UUID="aa0c38ed-3ed673d5-2a4fc139-3b18b48e" VERSION="4.0" MOUNT="/rom" TYPE="squashfs"
/dev/ubi0_1: UUID="639fb202-a764-4870-9a2a-73fe4309afbc" VERSION="w5r0" MOUNT="/overlay" TYPE="ubifs"
/dev/sda1: UUID="2565696e-9585-4fd4-a442-b6510b1d4fc9" LABEL="extroot" VERSION="1.0" TYPE="ext4"
/dev/sda2: UUID="dd37cee8-2310-43d1-95c9-511d06e0b67c" LABEL="swap" VERSION="1" TYPE="swap"
```

You should see an output like this. The LABEL for the partition `/dev/sda1` shoud be `extroot` and the type of the partition `/dev/sda2` should be swap.

## Configuring extroot

Configure the extroot mount entry.

```
DEVICE=/dev/sda1
eval $(block info ${DEVICE} | grep -o -e 'UUID="\S*"')
eval $(block info | grep -o -e 'MOUNT="\S*/overlay"')
uci -q delete fstab.extroot
uci set fstab.extroot="mount"
uci set fstab.extroot.uuid="${UUID}"
uci set fstab.extroot.target="${MOUNT}"
uci commit fstab
```

## Configuring the rootfs_data /ubifs

Configure a mount entry for the the original overlay.

```
ORIG="$(block info | sed -n -e '/MOUNT="\S*\/overlay"/s/:\s.*$//p')"
uci -q delete fstab.rwm
uci set fstab.rwm="mount"
uci set fstab.rwm.device="${ORIG}"
uci set fstab.rwm.target="/rwm"
uci commit fstab
```

 This will allow you to access the rootfs_data / ubifs partition and customize the extroot configuration /rwm/upper/etc/config/fstab.

 ## Transferring Data

Transfer the content of the current overlay to the external drive.

```
mount ${DEVICE} /mnt
tar -C ${MOUNT} -cvf - . | tar -C /mnt -xf -
```

## Enable Swap

Enable the swap partition.

```
SWAP_DEVICE=/dev/sda2
eval $(block info ${SWAP_DEVICE} | grep -o -e 'UUID="\S*"')

uci -q delete fstab.swap
uci set fstab.swap="swap"
uci set fstab.swap.uuid="${UUID}"
uci commit fstab
```

## Apply changes

Reboot the device to apply the changes.

```
reboot
```

## Testing

Run the following commands to verify if the things are alright.

Ensure the the `/dev/sda1` is mounted as overlay.

```
root@OpenWrt:~# mount | grep overlay
/dev/sda1 on /overlay type ext4 (rw,relatime)
overlayfs:/overlay on / type overlay (rw,noatime,lowerdir=/,upperdir=/overlay/upper,workdir=/overlay/work,uuid=on)
```

Ensure the the disk space is correct.

```
root@OpenWrt:~# df -h | grep overlay
/dev/sda1                13.2G      4.8M     12.5G   0% /overlay
overlayfs:/overlay       13.2G      4.8M     12.5G   0% /
```

Ensure swap is on.

```
root@OpenWrt:~# swapon -s
Filename				Type		Size		Used		Priority
/dev/sda2                               partition	1301500		0		-2
```

And it is visible in the output of `free` command.

```
root@OpenWrt:~# free
              total        used        free      shared  buff/cache   available
Mem:         249100       50388      166180         288       32532      157728
Swap:       1301500           0     1301500
```

The USB disk is now ready to be used.