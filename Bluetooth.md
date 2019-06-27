The Mk II includes a [u-blox ANNA-B112](https://www.u-blox.com/en/product/anna-b112-module)
Bluetooth module for out-of-band (in relation to USB interfaces) interaction
with a wireless client (e.g. mobile application).

# Documentation

  * [ANNA-B112 Datasheet](https://www.u-blox.com/sites/default/files/ANNA-B112_DataSheet_%28UBX-18011707%29.pdf)
  * [ANNA-B112 Protocol Specification](https://www.u-blox.com/sites/default/files/Bootloader_ProtocolSpec_%28UBX-17065404%29.pdf)
  * [u-connect AT Commands Manual](https://www.u-blox.com/sites/default/files/u-connect-ATCommands-Manual_%28UBX-14044127%29.pdf)
  * [u-blox Extended Data Mode](https://www.u-blox.com/sites/default/files/ExtendedDataMode_ProtocolSpec_%28UBX-14044126%29.pdf)
  * [u-blox Low Energy Serial Port Service](https://www.u-blox.com/sites/default/files/LowEnergySerialPortService_ProtocolSpec_%28UBX-16011192%29.pdf)
  * [u-blox resources](https://www.u-blox.com/en/product/anna-b112-module#tab-documentation-resources)

# Mobile apps

The following mobile applications are provided by u-blox for module evaluation:

  * iOS: [u-blox BLE](https://apps.apple.com/us/app/u-blox-ble/id575523395)
  * Android: [u-blox BLE](https://play.google.com/store/apps/details?id=com.ublox.BLE&hl=en)

# OpenCPU mode

The ANNA-B112 module supports an "OpenCPU" option to allow arbitrary firmware,
replacing the built-in u-blox one, on its Nordic Semiconductor nRF52832 SoC.
This allows provisioning of the SoC with Nordic SDK, Wirepas mesh, ARM Mbed or
arbitrary user firmware. The nRF52832 SoC features an ARM Cortex-M4 CPU with
512 kB of internal Flash and 64 kB of RAM.

  * [nRF528832 Datasheet](https://www.nordicsemi.com/-/media/Software-and-other-downloads/Product-Briefs/nRF52832-product-brief.pdf?la=en)
