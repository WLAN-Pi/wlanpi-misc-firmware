# MT7961 Bluetooth firmware notes

This package ships `mediatek/BT_RAM_CODE_MT7961_1_2_hdr.bin` and no longer
deletes it during `postinst`. The notes below collect the referenced upstream
discussion and the observations made on a WLAN Pi M4+ test unit. No root cause
is asserted here.

## Referenced discussion

- morrownr/USB-WiFi issue #478, "[Help]: Alfa AWUS036AXM device not recognized"
  <https://github.com/morrownr/USB-WiFi/issues/478> (closed 2024-10-03).
- Kernel bugzilla #219040 (linked from #478).
- linux-bluetooth / linux-mediatek thread, "Bluetooth: btmtk: Remove resetting
  mt7921 before downloading the fw" (Hao Qin, Aug 2024).

Selected statements from that thread, attributed to their authors:

- morrownr: the workaround of deleting the Bluetooth firmware ("evidently this
  only shows up on AMD systems") was proposed as a test, not a fix.
- ZerBea: "Even with a workaround (disable BT, remove BT firmware), the driver
  sooner or later dies."
- MediaTek (Chris Lu): "the firmware we push to Kernel is not for
  'MT7921AUN', software, firmware and hardware may not match. Using that dongle
  with Linux-Based Notebook may lead to some unexpected errors such as
  Bluetooth can't setup successfully." MediaTek stated a firmware-side change
  was planned for November 2024.
- The kernel regression tracker thread references commit `ccfc8948d7e4d9`
  ("Bluetooth: btusb: mediatek: reset the controller before downloading the
  fw") and the follow-up patch that removes MT7921 from that reset rule.

## Test unit

- WLAN Pi M4+ (CM4), `7.1.12-v8-wlanpi+`, Debian 13 (trixie),
  `wlanpi-misc-firmware` 1.0.18.
- Adapter: MediaTek MT7961 combo, USB `0e8d:7961`, interfaces `1-1.1:1.0` and
  `1-1.1:1.1` (`btusb`) plus `1-1.1:1.3` (`mt7921u`), connected through a
  bus-powered Terminus USB 2.0 hub on the CM4 `dwc2` controller.
- MT7961 Wi-Fi firmware on the unit at test time: `20251223091050a`
  (`WIFI_MT7961_patch_mcu_1_2_hdr.bin`), `20251223091148` (reported WM firmware).

### BT firmware absent (shipped 1.0.18 state)

`dmesg` showed a repeating sequence, roughly every two seconds:

```
mt7921u 1-1.1:1.3: HW/SW Version: 0x8a108a10, Build Time: 20251223091050a
mt7921u 1-1.1:1.3: WM Firmware Version: ____010000, Build Time: 20251223091148
usb 1-1.1: reset high-speed USB device number 3 using dwc2
bluetooth hci0: Direct firmware load for mediatek/BT_RAM_CODE_MT7961_1_2_hdr.bin failed with error -2
Bluetooth: hci0: Failed to load firmware file (-2)
Bluetooth: hci0: Failed to set up firmware (-2)
```

`/sys/class/ieee80211/` cycled: the MT7961 phy appeared briefly, then
disappeared.

### BT firmware present

With `BT_RAM_CODE_MT7961_1_2_hdr.bin` (build `20260224111243`) placed in
`/usr/lib/firmware/mediatek/`:

```
Bluetooth: hci0: HW/SW Version: 0x008a008a, Build Time: 20260224111243
Bluetooth: hci0: command 0xfd98 tx timeout
Bluetooth: hci0: Failed to apply iso setting (-110)
Bluetooth: hci0: Opcode 0x0c03 failed: -110
```

The USB reset messages stopped during the observation window. The MT7961 Wi-Fi
interface registered as `wlan1` (`0e8d:7961` on `1-1.1:1.3`) and an
`iw dev wlan1 scan` returned APs. The Bluetooth stack continued to report
`-110` command timeouts.

### Later observation

After continued testing with all three adapters active, `1-1.1` dropped off the
bus:

```
usb 1-1.1: device not accepting address 3, error -110
usb 1-1.1: USB disconnect, device number 3
usb 1-1.1: device descriptor read/64, error -110
```

The adapter did not re-enumerate until it was physically re-plugged. Power
(bus-powered hub, multiple adapters on one USB 2.0 controller) was not
isolated.

### AXE3000 (0846:9060) observation

At baseline the Netgear AXE3000 bound to `mt7921u` and registered a wiphy
(`/sys/class/ieee80211/phy2`) with no interface under
`/sys/class/ieee80211/phy2/device/net/`. After the MT7961 stopped cycling, the
AXE3000 registered an interface (`94:18:65:...`) and an `iw dev` scan returned
APs, with the MT7961 Wi-Fi firmware still at the 1.0.18 versions.

## Scope

The above are observations from a single unit and a single kernel. They are
recorded here for reference only; this document does not assert a root cause or
a causal relationship between any of the observed events.
