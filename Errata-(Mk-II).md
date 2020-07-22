USB armory Mk II (rev. β)
=========================

Errata: speed class limitation for external uSD and internal eMMC
-----------------------------------------------------------------

The SD and NAND SoC blocks, as well as the associated cards, receive a 3.3V
signaling voltage, this prevents use of higher speed SD/MMC bus modes that
require 1.8V signaling.

For this reason the maximum achievable speeds are High Speed mode (25MB/s,
50MHz, 3.3V) for external uSD and High Speed DDR mode (104MB/s, 52MHz, 3.3V)
for the internal eMMC.

Future USB armory Mk II revisions will integrate the following changes:

  * eMMC: signaling voltage set to 1.8V to support HS200 mode
    (200MB/s, 200MHz, 1.8V but at 150MB/s due to NXP ERR010450).

  * uSD: switchable 1.8V/3.3V signaling voltage to support SDR104
    (104MB/s, 208MHz, 1.8V).

Errata: inverted UART RTS/CTS signals (resolved with workaround)
----------------------------------------------------------------

The UART1 and UART2 hardware flow control RTS/CTS signals, routed respectively
from the ANNA-B112 BLE module and the debug accessory connection towards the
SoC, are inverted.

This prevents automatic hardware flow control when sending/receiving data from
either the BLE module or the debug accessory FT4232H converter.

For UART1 the issue does not affect standard uses of the BLE module as, even at
highest rates, the Linux i.MX serial driver is buffered adequately and never
suffers from RX FIFO overruns.

The lack of flow control might become evident only on high data rate BLE
Extended Data Mode (EDM) transfers, executed by polling (e.g. without
interrupts) and without buffering.

In such occurrences the issue can be worked around by using RTS/CTS as GPIOs to
enforce correct direction and drive them by software, bypassing the UART
controller automatic flow control.

The UART2 is accessible through the USB armory Mk II
[debug accessory](https://github.com/f-secure-foundry/usbarmory/tree/master/hardware/mark-two-debug-accessory)
in UART mode, when accessing it ensure that hardware flow control is disabled
(default with the Linux FTDI driver and most terminal applications).

Future USB armory Mk II revisions will swap UART1 and UART2 RTS/CTS to address
the issue.

Errata: unreliable USB Serial Downloader (resolved with workaround)
-------------------------------------------------------------------

The Serial Downloader mode checks first for activity on UART1/UART2, then on
the USB OTG1 interface. When activity is detected on UART1/UART2 the USB
interface is not configured for serial download.

On some devices when booting in Serial Downloader mode (e.g. without a valid
boot media) the device detects spurious activity on UART1 caused by the BLE
module at startup, this prevents configuration of Serial Downloader mode over
USB and results of the following messages on the host (Linux example shown):

```
kernel: usb 2-2.3: new high-speed USB device number 94 using xhci_hcd
kernel: usb 2-2.3: device descriptor read/64, error -110
```

The issue is worked around by disabling the UART Serial Downloader mode,
leaving only the USB mode active, by fusing the `UART_SERIAL_DOWNLOAD_DISABLE`
OTP fuse (example with [crucible](https://github.com/f-secure-foundry/crucible) tool
shown):

```
crucible -m IMX6ULZ -r 0 -b 16 -e big blow UART_SERIAL_DOWNLOAD_DISABLE 1
```

Errata: unreliable serial connection to BLE module (resolved with workaround)
-----------------------------------------------------------------------------

On a relevant percentage of pre-production boards (~25%) the serial connection
to the BLE module is unreliable, suggesting jitter with clock settings.

The ANNA-B112 module features an automatic detection procedure for low
frequency clock source, whether to use the internal RC oscillator or an
external crystal.

This procedure does not work reliably when the `XL_1` and `XL_2` pads are left
unconnected, despite this being consistent with the u-blox own design
guidelines.

In order to force the internal RC oscillator as internal LF clock source the
`AT+UPRODLFCLK=0,16,2` command can be used, however in some cases the serial
interface does not work reliably enough to allow such command to be issued.

Therefore the only reliable measure remains patching, via OpenOCD, the module
flash to manually overwrite the `CUSTOMER[31]` register with the desired
setting.

Both methods are implemented in the `ble rc_lfck (flash|at)` command of the
[armoryctl](https://github.com/f-secure-foundry/armoryctl) tool.

Following our bug report, u-blox recommends grounding of the `XL_1` and `XL_2`
pads in ANNA-B112 System Integration Manual revisions R06 or later.

Future USB armory Mk II revisions will ground `XL_1` and `XL_2`.

Errata: instability with specific Linux kernel versions (resolved on <= 4.19, >= 5.3)
-------------------------------------------------------------------------------------

The following commit introduces CPU/memory instability (kernel versions > 4.9):
  http://archive.lwn.net:8080/devicetree/1535701998-20443-2-git-send-email-Anson.Huang@nxp.com/t/#m37a316952c72a0b8d2ae16e7d0720a498048409c

It is not clear why P1 IPG clock is required on i.MX6UL MMDC block as the
reference manual does not reference it.

The instability is addressed by either reverting the earlier commit or applying
the following one:
  https://patchwork.kernel.org/patch/10950145/

The patch is included in kernels >= 5.3,

Errata: glitch on CPU frequency changes (resolved)
--------------------------------------------------

When the Linux CPU governor is set `ondemand`, and on high CPU loads, the
system is highly unstable on a relevant percentage of pre-production boards
(~40%).

The instability is resolved by forcing the CPU governor to `performance`.

The issue appears similar to what reported at the following URL, which also
includes a utility which reliably triggers the issue:
  https://github.com/imx6-dongle/linux-imx/issues/9

The following blog post higlights a similar issue which might be "appearing as
if the problem stemmed from a CPU frequency scaling problem.":
  https://boundarydevices.com/i-mx-6dq-u-boot-updates/

The issue has been evaluated as the result of tight DDR calibration values,
using a different round of DDR calibration resolved the issue:
  https://github.com/f-secure-foundry/usbarmory/commit/d3f98bbc812c619e6d1d4136b53806169ea2d34e

Errata: charger detection issues
--------------------------------

According to NXP errata ERR006281 the charger detection feature must be
disabled, otherwise a poor DP signal causes a reset of the USB connection. This
is already performed in vanilla bootloaders (U-Boot).

Such feature is triggered when using the USB armory Mk II on Type-A ports using
a USB-C to Type-A adapter.

Under Linux such behavior causes, on rare cases such as when using the
right-hand side USB port of a Thinkpad x250, a reset when done using the
`USB_ANALOG_USB1_CHRG_DETECT` register.

The solution is to perform the operation through
`USB_ANALOG_USB1_CHRG_DETECT_SET`, `USB_ANALOG_USB1_CHRG_DETECT_CLR`.

It is unclear while under U-Boot, which does not use the SET/CLR registers,
this works.

The symptom can also be eliminated by commenting function
`imx_anatop_usb_chrg_detect_disable` in Linux kernel source file
`arch/arm/mach-imx/anatop.c`.

On some Thinkpad x250 right-hand side USB ports none of those mitigations
appear to work, suggesting a different/additional issue.

It should be noted that when loading code through `imx_usb` the issue does not
appear, it is speculated that the USB initialization performed by the boot ROM
itself (due to Serial Download Protocol mode) prevents the issue.

The issue remains unsolved as it has only been encountered on a single laptop,
only on one of its ports and while using a Type-A adapter.

Errata: Type-C plug/receptacle reset (plug: resolved, receptacle: workaround)
-----------------------------------------------------------------------------

The FUSB303 Type-C port Controller, when in I2C mode, requires the ENABLE bit
set in CONTROL1 register, in order to enable its operation and consequently
allow the following:

  * reading the amount of current made available by the host
  * detecting a debug accessory

On a test Apple Macbook Pro setting the ENABLE bit results in a port reset by
the host. This makes the FUSB303 incompatible with "dead battery" operation
when attached as a power sink.

To fix the problem the TUSB320 P/N has been considered as it is almost pin
compatible with the FUSB303, however the TUSB320 is not ideal for use as
receptacle port controller.

Specifically the TUSB320 does not allow debug accessory mode detection in UFP
mode, given that the debug accessory is configured as power source this makes
the TUSB320 incompatible with it.

The TUSB320LI variant would solve the debug accessory incompatibility but
nonetheless, due to the lack of a dedicated DEBUG output (available in the
FUSB303), it would require the debug mode switch (U18) to be operated by
software rather than automatically, which is inconvenient.

The same issue applies to the FUSB301A which, while it does update status and
type registers when attached in dead battery mode, it does not feature a
dedicated DEBUG output.

For all such reasons only the plug FUSB303 Type-C port controller has been
replaced with the TUSB320, to allow host current read which is only required on
this side as the plug role is forced to UFP/sink role.

The receptacle port keeps the FUSB303 to ensure correct debug accessory
functionality, without requiring bootloader and firmware to be aware of its
operation, as the majority of receptacle port use cases would not involved a
sink connection to a host.

In cases where users desire a sink connection to a host, using the receptacle
port, the default firmware behavior can be overridden by software as the ENABLE
command issued by the bootloader can be disabled.

Useful links:  
https://e2e.ti.com/support/interface/f/138/p/707926/2608914  
https://e2e.ti.com/support/interface/f/138/t/813069

Manual enable:
`i2cset -y 0 0x31 0x5 0xbb` or
 [armoryctl](https://github.com/f-secure-foundry/armoryctl) `fusb (enable|disable)`
