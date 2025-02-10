
# Installing OpenWRT Firmware

*Please note that this needs to be done only once. The router may have this firmware already installed.*

The OpenWRT Firmware can be downloaded using the [firmware-selector of openwrt](https://firmware-selector.openwrt.org/?version=24.10.0&target=ramips%2Fmt7621&id=dlink_dir-853-a3). You need to download the factory image.

The Latest version as of Feb 2025 is linked below.

[OpenWrt firmware version 24.10 for model D-Link DIR-853 A3][1].

[1]: https://downloads.openwrt.org/releases/24.10.0/targets/ramips/mt7621/openwrt-24.10.0-ramips-mt7621-dlink_dir-853-a3-squashfs-factory.bin

Every router will have a way to update firmware. It is available from the admin interface of the router, typically in the _System_ tab/menu and it will be labelled as _Backup / update firmware_.

Once you select the factory image and upload, the led lights on router will flash repeated indicating that it is resetting and after a minute or so, the router will be ready with openwrt.
