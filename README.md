# wlanpi-misc-firmware

Packaging miscellaneous firmware files, like MediaTek, Qualcomm ath12k and
Ralink. Goal is to have a package with newer or patched firmware files than the
ones currently available from official repositories. The package also absorbs
the wireless-relevant contents of `firmware-mediatek` (which it
Conflicts/Replaces), so removing that package causes no regression.

Firmware repositories:

- MediaTek:
  <https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/mediatek>
- Qualcomm ath12k:
  <https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/tree/ath12k>

## Contents

| Chipset | Files |
| --- | --- |
| MediaTek MT7610U | `mediatek/mt7610u.bin`, `mediatek/mt7610e.bin` |
| MediaTek MT7612U | `mt7662.bin`, `mt7662_rom_patch.bin` |
| MediaTek MT7961 (MT7921) | `mediatek/WIFI_MT7961_patch_mcu_1_2_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7961_1.bin`, `mediatek/BT_RAM_CODE_MT7961_1_2_hdr.bin` |
| MediaTek MT7922 | `mediatek/WIFI_MT7922_patch_mcu_1_1_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7922_1.bin`, `mediatek/BT_RAM_CODE_MT7922_1_1_hdr.bin` |
| MediaTek MT7925 (Wi-Fi 7) | `mediatek/mt7925/WIFI_MT7925_PATCH_MCU_1_1_hdr.bin`, `mediatek/mt7925/WIFI_RAM_CODE_MT7925_1_1.bin`, `mediatek/mt7925/BT_RAM_CODE_MT7925_1_1_hdr.bin` |
| MediaTek MT7927 (Wi-Fi 7, Filogic 380) | `mediatek/mt7927/WIFI_MT6639_PATCH_MCU_2_1_hdr.bin`, `mediatek/mt7927/WIFI_RAM_CODE_MT6639_2_1.bin` (WiFi only; Bluetooth firmware not yet available upstream) |
| MediaTek MT7902 (Wi-Fi 6E) | `mediatek/WIFI_MT7902_patch_mcu_1_1_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7902_1.bin`, `mediatek/BT_RAM_CODE_MT7902_1_1_hdr.bin` |
| MediaTek MT7961/MT7921 1a variants | `mediatek/WIFI_MT7961_patch_mcu_1a_2_hdr.bin`, `mediatek/WIFI_RAM_CODE_MT7961_1a.bin`, `mediatek/BT_RAM_CODE_MT7961_1a_2_hdr.bin` |
| MediaTek MT7601U | `mediatek/mt7601u.bin`, `mt7601u.bin` |
| MediaTek MT7650/MT7662/MT7662U | `mediatek/mt7650.bin`, `mediatek/mt7650e.bin`, `mediatek/mt7662.bin`, `mediatek/mt7662_rom_patch.bin`, `mediatek/mt7662u.bin`, `mediatek/mt7662u_rom_patch.bin`, `mt7650.bin`, `mt7662.bin`, `mt7662_rom_patch.bin` |
| MediaTek MT7615/MT7622/MT7663/MT7668 | `mediatek/mt7615_cr4.bin`, `mediatek/mt7615_n9.bin`, `mediatek/mt7615_rom_patch.bin`, `mediatek/mt7622_n9.bin`, `mediatek/mt7622_rom_patch.bin`, `mediatek/mt7622pr2h.bin`, `mediatek/mt7663_n9_rebb.bin`, `mediatek/mt7663_n9_v3.bin`, `mediatek/mt7663pr2h.bin`, `mediatek/mt7663pr2h_rebb.bin`, `mediatek/mt7668pr2h.bin` |
| MediaTek MT7915/MT7916 | `mediatek/mt7915_eeprom.bin`, `mediatek/mt7915_eeprom_dbdc.bin`, `mediatek/mt7915_rom_patch.bin`, `mediatek/mt7915_wa.bin`, `mediatek/mt7915_wm.bin`, `mediatek/mt7916_eeprom.bin`, `mediatek/mt7916_rom_patch.bin`, `mediatek/mt7916_wa.bin`, `mediatek/mt7916_wm.bin` |
| Ralink rt2x00 | `rt73.bin`, `rt2561.bin`, `rt2561s.bin`, `rt2661.bin`, `rt2860.bin`, `rt2870.bin`, `rt3071.bin`, `rt3290.bin` |
| Qualcomm WCN7850/QCN7851 (Wi-Fi 7, e.g. QCNCM865) | `ath12k/WCN7850/hw2.0/amss.bin`, `ath12k/WCN7850/hw2.0/board-2.bin`, `ath12k/WCN7850/hw2.0/m3.bin`, `ath12k/WCN7850/hw2.0/ncm865/amss.bin`, `ath12k/WCN7850/hw2.0/ncm865/m3.bin` |

Some drivers request their firmware from the top level of `/usr/lib/firmware`
(without a vendor prefix), so `mt7662.bin`, `mt7662_rom_patch.bin`, `mt7601u.bin`,
`mt7650.bin` and the `rt*.bin` files are additionally installed at
`/usr/lib/firmware/` (the `mediatek/` copies cover newer drivers). All other
MediaTek files are installed under `/usr/lib/firmware/mediatek/` and the ath12k
files under `/usr/lib/firmware/ath12k/`.

## Diversions

The package `Conflicts:`/`Replaces:` `firmware-mediatek` and uses `dpkg-divert`
in `preinst`/`postrm` so its files take precedence over any copies shipped by
the distro firmware packages.

## Requirements

- MediaTek MT7927 (PCI ID 14c3:7927, Filogic 380) requires a kernel with an
  MT7927-capable mt76 driver (upstream support landed via the mt7925e driver;
  verified working on kernel 7.3.0-wlanpi). MT7927 Bluetooth firmware is not
  yet available in linux-firmware.
- Qualcomm QCNCM865/WCN7850 requires the ath12k driver with WCN7850 support.

## Updating firmware

1. Pick a `linux-firmware` tag.
2. Download the target files from `mediatek/` (and the `mediatek/mt7925/`,
   `mediatek/mt7927/` subdirectories) and `ath12k/` at that tag.
3. Copy them into the matching path under `lib/firmware/`, keeping the
   upstream directory layout.
4. Verify the version strings:
   `strings <file> | grep -oE '20[0-9]{12}[a-z]?' | sort -u | tail -1`
   (ath12k: `strings amss.bin | grep QC_IMAGE_VERSION_STRING`)
5. Record the old and new versions in `debian/changelog`.
6. If a new file is added, update `debian/wlanpi-misc-firmware.install`,
   `debian/control` and the `preinst`/`postrm` diversion lists.

The MediaTek files are currently from the `20261019` tag (MT7925 from
`20261019-23`/`0a62bdca`) and the ath12k files from `20261019`. See issue #20
for the verification that is still outstanding for the MT7922/MT7925 blobs.

## Building

```
dpkg-buildpackage -us -uc -b
```

The package targets Debian trixie.
