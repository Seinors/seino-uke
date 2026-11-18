# mozzrath kernel for Xiaomi Pad 7 (`uke`)

Custom Linux 6.1 kernel for the Xiaomi Pad 7 / POCO Pad X1 (`uke`, SM7675),
built from [Xiaomi-Pad-7-Development/android_kernel_xiaomi_sm8635](https://github.com/Xiaomi-Pad-7-Development/android_kernel_xiaomi_sm8635)
(branch `sixteen`, Linux 6.1.182) using the device's own kernel configuration
(`configs/uke_gki_defconfig`, extracted from `/proc/config.gz`).

Only the kernel `Image` is built. The stock ramdisk, DTB and vendor modules are
kept, so vendor module compatibility relies on `CONFIG_MODVERSIONS` KMI CRC
matching (same approach used by other uke kernel projects).

## Build

GitHub Actions -> **mozzrath-kernel** -> Run workflow:

- `perf=false`: base build, stock configuration (minus kernel trimming/debug info).
- `perf=true`: adds `-O3` (`CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE_O3`) and BBR,
  mirroring Xiaomi's own `uke_perf.config`.

Artifacts: `mozzrath-Image`, `mozzrath.config`, `mozzrath.System.map`,
`utsrelease.h`.

## Flash (slot B)

Back up first, always:

```sh
su -c 'dd if=/dev/block/by-name/boot_b of=/sdcard/boot_b_stock_mozzrath.img'
sha256sum /sdcard/boot_b_stock_mozzrath.img
```

Repack the stock boot image with the new kernel (Magisk is in `init_boot_b`,
so it is not touched):

```sh
mkdir -p ~/mozzrath && cd ~/mozzrath
cp /sdcard/boot_b_stock_mozzrath.img .
magiskboot unpack boot_b_stock_mozzrath.img
cp /path/to/mozzrath-Image kernel
magiskboot repack boot_b_stock_mozzrath.img boot_b_mozzrath.img
```

Flash with fastboot from a PC:

```sh
adb pull /sdcard/boot_b_mozzrath.img .
adb reboot bootloader
fastboot flash boot_b boot_b_mozzrath.img
fastboot reboot
```

Rollback:

```sh
fastboot flash boot_b boot_b_stock_mozzrath.img
```

## Verify

```sh
uname -r
dmesg | grep -Ei 'disagrees|version magic|Unknown symbol|exec format'
lsmod | head
```

Smoke-test Wi-Fi, Bluetooth, display, touch, pen, audio, cameras and charging.

## Notes

- The active vbmeta has AVB verification and hashtree disabled, so a custom
  `boot` image boots even though the bootloader reports a locked state.
- Vendor modules are signed by Xiaomi; `CONFIG_MODULE_SIG_FORCE` is unset, so
  loading them with a differently signed kernel only taints the kernel.
- HyperOS/crDroid OTA updates overwrite `boot_b`; reflash after updating.
