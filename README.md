# wlanpi-misc-firmware

Packaging miscellaneous firmware files, like MediaTek. Goal is to have a package
with newer or patched firmware files than the ones currently available from
official repositories.

MediaTek firmware repository:
<https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/mediatek>

## Contents

| Chipset | Files |
| --- | --- |
| MediaTek MT7610U | `mediatek/mt7610u.bin`, `mediatek/mt7610e.bin` |
| MediaTek MT7612U | `mt7662.bin`, `mt7662_rom_patch.bin` |
| MediaTek MT7961 (MT7921) | `mediatek/WIFI_MT7961_patch_mcu_1_2_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7961_1.bin`, `mediatek/BT_RAM_CODE_MT7961_1_2_hdr.bin` |
| MediaTek MT7922 | `mediatek/WIFI_MT7922_patch_mcu_1_1_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7922_1.bin`, `mediatek/BT_RAM_CODE_MT7922_1_1_hdr.bin` |
| MediaTek MT7925 (Wi-Fi 7) | `mediatek/mt7925/WIFI_MT7925_PATCH_MCU_1_1_hdr.bin`, `mediatek/mt7925/WIFI_RAM_CODE_MT7925_1_1.bin`, `mediatek/mt7925/BT_RAM_CODE_MT7925_1_1_hdr.bin` |

The MT7612U driver requests `mt7662.bin` and `mt7662_rom_patch.bin` from the top
level of `/usr/lib/firmware` (without the `mediatek/` prefix), so those two files are
installed at `/usr/lib/firmware/`. All other files are installed under
`/usr/lib/firmware/mediatek/`.

## Diversions

The package `Conflicts:`/`Replaces:` `firmware-mediatek` and uses `dpkg-divert`
in `preinst`/`postrm` so its files take precedence over any copies shipped by
the distro firmware packages.

## Updating firmware

1. Pick a `linux-firmware` tag.
2. Download the target files from `mediatek/` (and `mediatek/mt7925/` for
   MT7925) at that tag.
3. Copy them into `lib/firmware/mediatek/` (MT7922/MT7961/MT7610U) or
   `lib/firmware/mt7662*.bin` (MT7612U).
4. Verify the version strings:
   `strings <file> | grep -oE '20[0-9]{12}[a-z]?' | sort -u | tail -1`
5. Record the old and new versions in `debian/changelog`.
6. If a new file is added, update `debian/wlanpi-misc-firmware.install`,
   `debian/control` and the `preinst`/`postrm` diversion lists.

The MT7922/MT7925 files are currently from the `20260916` tag. See issue #20 for
the verification that is still outstanding for those blobs.

## Building

```
dpkg-buildpackage -us -uc -b
```

The package targets Debian trixie.
