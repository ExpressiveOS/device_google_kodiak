# Pixel 11 Pro XL (kodiak) for ExpressiveOS

Device tree for Google Pixel 11 Pro XL (`kodiak`, Tensor G6 `malibu`, GKI
6.12). Vendor, kernel and firmware come from the stock factory image
`CP4A.261105.040.D1` (Android 17 QPR2, November 2026); the product and system_ext
blobs in proprietary-files.txt are still the `CD1A.261005.003` ones.

Started from `aospdtgen 1.2.1` output, then restructured after LineageOS's
Tensor trees (`device/google/laguna`, `device/google/muzel`).

## Layout

| Image | Source |
| --- | --- |
| system, system_ext, product | built (ExpressiveOS) |
| init_boot (generic ramdisk) | built |
| vendor_boot (first-stage fstab, recovery) | built |
| vbmeta, vbmeta_system, vbmeta_vendor | built (verification disabled: `--flags 3`) |
| boot, vendor_kernel_boot | Espada kernel (Linux 6.12.92, CASS + Simple LMK, plain flavor) by Kevtrejo37; stock GKI fallback |
| dtbo | stock |
| pvmfw | built (AOSP) |
| vendor, vendor_dlkm, system_dlkm | stock |
| bootloader/modem firmware | stock, flashed raw by the OTA |

Google publishes no malibu vendor sources (SELinux policy, HAL configs), so
vendor stays stock for now.

## Kernel Credits

- **Espada Kernel** by **Kevtrejo37** ([XDA Thread](https://xdaforums.com/t/4799975/), [Telegram](https://t.me/+DJmYe5_dfmwyODUx), [GitHub](https://github.com/atrejokm301/espada-kernel)). Features CASS scheduler, Simple LMK, measured EM, and upstream Linux 6.12 patches.
- ZRAM read-ahead fix by **The GrapheneOS Project**.
- Sultan Alsawaf for CASS and Simple LMK.


## Building

ExpressiveOS builds this tree with its build kit ([`expressiveos-cheetah/`](https://github.com/denson9874/Denson9874-Device-Central/tree/main/expressiveos-cheetah)),
in the source tree ExpressiveOS's own manifest syncs (`~/expressiveos`, set up by the kit's
`setup.sh`). The kit's `devices/kodiak.conf` holds what's particular to this phone: the lunch
target `lineage_kodiak-cp2a-user`, the model, the release folders, and where its kernel, vendor and
firmware come from.

```bash
EOS_DEVICE=kodiak ~/expressiveos-cheetah/tools/eos-images.sh   # the build
EOS_DEVICE=kodiak ~/expressiveos-cheetah/tools/eos-pack.sh     # the fastboot zip
```

From the runner, that's the action `expressiveos-build` with query `device=kodiak`.

The manifest doesn't sync the two folders of Google's files this tree needs. The kit's
`prepare.sh` puts them in place before each build:

- `vendor/google/kodiak-images`, whole stock partition images (vendor, vendor_dlkm,
  system_dlkm, boot, dtbo, firmware), made by this tree's `extract-images.sh` from the newest
  stock image zip on the build server (`~/android-dt/firmware/image-kodiak-*.zip`, from
  [Google's factory image](https://developers.google.com/android/images)), again only when
  that zip changes. It isn't published.
- `vendor/google/kodiak`, the proprietary blobs from `proprietary-files.txt`:
  [vendor_google_kodiak](https://github.com/denson9874/vendor_google_kodiak), or regenerate it
  with `./extract-files.py <dump>`. The build server keeps its own copy.

The kit builds the `user` variant. `userdebug` mounts debugfs early, which the stock vendor
policy denies, and the display driver can panic on boot.
