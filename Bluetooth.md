The Mk II includes a [u-blox ANNA-B112](https://www.u-blox.com/en/product/anna-b112-module)
Bluetooth module for out-of-band (in relation to USB interfaces) interaction
with a wireless client (e.g. mobile application).

The module SoC is a Nordic Semiconductor nRF52832, supporting low energy (BLE)
Bluetooth 5 and Bluetooth mesh.

# Documentation

  * [ANNA-B112 Datasheet](https://www.u-blox.com/sites/default/files/ANNA-B112_DataSheet_%28UBX-18011707%29.pdf)
  * [ANNA-B112 Protocol Specification](https://www.u-blox.com/sites/default/files/Bootloader_ProtocolSpec_%28UBX-17065404%29.pdf)
  * [ANNA-B112 System Integration Manual](https://www.u-blox.com/sites/default/files/ANNA-B112_SIM_%28UBX-18009821%29.pdf)
  * [u-connect AT Commands Manual](https://www.u-blox.com/sites/default/files/u-connect-ATCommands-Manual_%28UBX-14044127%29.pdf)
  * [u-blox Extended Data Mode](https://www.u-blox.com/sites/default/files/ExtendedDataMode_ProtocolSpec_%28UBX-14044126%29.pdf)
  * [u-blox Low Energy Serial Port Service](https://www.u-blox.com/sites/default/files/LowEnergySerialPortService_ProtocolSpec_%28UBX-16011192%29.pdf)
  * [u-blox resources](https://www.u-blox.com/en/product/anna-b112-module#tab-documentation-resources)

# Mobile apps

The following mobile applications are provided by u-blox for module evaluation:

  * iOS: [u-blox BLE](https://apps.apple.com/us/app/u-blox-ble/id575523395)
  * Android: [u-blox BLE](https://play.google.com/store/apps/details?id=com.ublox.BLE&hl=en)

# Control tool

The [armoryctl](https://github.com/f-secure-foundry/armoryctl) can be used to perform some of the procedures illustrated below.

# Serial connection

All serial AT commands can be issued with your favorite terminal program
against device `/dev/ttymxc0` and baud rate 115200, example:

```
minicom -b 115200 -D /dev/ttymxc0
```

# Initial configuration

A one-time configuration, to force the internal RC oscillator as low frequency
clock source, must be performed through the following AT commands:

```
AT+UPROD=1
AT+UPRODLFCLK=0,16,2
```

This can also be accomplished with [armoryctl](https://github.com/f-secure-foundry/armoryctl) `ble rc_lfck (flash|at)` command.

# Toggling visibility

The following AT commands set BLE as non discoverable, non pairable, non
connectable and disable any role (central or peripheral).

```
AT+UBTDM=1
AT+UBTPM=1
AT+UBTCM=1
AT+UBTLE=0
```

The following commands allow permanently store the configuration

```
AT&W
AT+CPWROFF
```

This can also be accomplished with [armoryctl](https://github.com/f-secure-foundry/armoryctl) `ble (enable|disable)` commands.

# Firmware update

The following procedure allows to update the u-blox connectivity software
(u-connect).

This can also be accomplished with [armoryctl](https://github.com/f-secure-foundry/armoryctl) `ble update` command.

## Check the running firmware version

```
AT+GMR
```

## Download firmware updates

The ANNA-B112 firmware is available on the
[module product page](https://www.u-blox.com/en/product/anna-b112-u-connect).

The firmware zip archive contains the following files under the `uart`
subdirectory, which are used during the update:

 * Nordic Semiconductors Bluetooth stack: `s132_nrf52_x.x.x_softdevice.bin`
 * u-connect application:  `ANNA-B112-SW-x.x.x.bin`
 * update metadata: `ANNA-B112-Configuration-x.x.x.json`

## Enter bootloader mode

```
# configure GPIOs
echo "9" > /sys/class/gpio/export
echo "26" > /sys/class/gpio/export
echo "27" > /sys/class/gpio/export
echo "out" > /sys/class/gpio/gpio9/direction
echo "out" > /sys/class/gpio/gpio26/direction
echo "out" > /sys/class/gpio/gpio27/direction
# set SWITCH_1 and SWITCH_2 to low
echo "0" > /sys/class/gpio/gpio27/value
echo "0" > /sys/class/gpio/gpio26/value
# toggle RESET_N
echo "0" > /sys/class/gpio/gpio9/value
echo "1" > /sys/class/gpio/gpio9/value
```

## Update the Nordic Semiconductors Bluetooth stack

The `SoftDevice` section of the update metadata contains the `Address`, `Size`,
and `Crc32` values to be used in the following commands:

1. Set the bootloader in xmodem erase+program mode at the relevant address:

```
x <Address>
```

2. Send the file `s132_nrf52_x.x.x_softdevice.bin` using xmodem transfer mode,
   with minicom this can be done using shortcut Ctrl+a+S.

6. Finalize the update by verifying size and checksum values:

```
c SOFTDEVICE <Size> <Crc32>
```

## Update the u-connect application

The `ConnectivitySoftware` section of the update metadata contains the
`Address` value to be used in the following commands:

1. Set the bootloader in xmodem erase+program mode at the relevant address:

```
x <Address>
```

2. Send the file `ANNA-B112-SW-x.x.x.bin` using xmodem transfer mode, with
minicom this can be done using shortcut Ctrl+a+S.

## Exit bootloader mode

```
# set SWITCH_1 and SWITCH_2 to high
echo "1" > /sys/class/gpio/gpio27/value
echo "1" > /sys/class/gpio/gpio26/value
# toggle RESET_N
echo "0" > /sys/class/gpio/gpio9/value
echo "1" > /sys/class/gpio/gpio9/value
```

# OpenCPU mode

The ANNA-B112 module supports an "OpenCPU" option to allow arbitrary firmware,
replacing the built-in u-blox one, on its Nordic Semiconductor nRF52832 SoC.
This allows provisioning of the SoC with Nordic SDK, Wirepas mesh, ARM Mbed or
arbitrary user firmware. The nRF52832 SoC features an ARM Cortex-M4 CPU with
512 kB of internal Flash and 64 kB of RAM.

  * [nRF528832 Datasheet](https://www.nordicsemi.com/-/media/Software-and-other-downloads/Product-Briefs/nRF52832-product-brief.pdf?la=en)

## Using OpenOCD for JTAG access

The following instructions have been tested on the USB armory Mk II using its
[standard Debian image](https://github.com/f-secure-foundry/usbarmory-debian-base_image).

1. Download and compile OpenOCD from its github repository:

```
git clone https://github.com/ntfreak/openocd
cd openocd
./bootstrap
./configure --disable-internal-libjaylink --disable-jlink --enable-imx_gpio
make
```

2. Create the `ANNA-B112.cfg` configuration file with the following contents:

```
transport select swd
source [find target/nrf52.cfg]
```

4. Create the interface file as follows:

```
cp ./tcl/interface/imx-native.cfg usbarmory-mark-two.cfg
sed -i -e 's/imx_gpio_swd_nums 1 6/imx_gpio_swd_nums 4 6/' usbarmory-mark-two.cfg
```

3. Launch OpenOCD

```
sudo ./src/openocd --search ./tcl -f usbarmory-mark-two.cfg -f ANNA-B112.cfg
```

The output should be similar to the following one:

```
Open On-Chip Debugger 0.10.0+dev-00924-g16496488 (2019-08-09-12:39)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : imx_gpio GPIO JTAG/SWD bitbang driver
Info : SWD only mode enabled (specify tck, tms, tdi and tdo gpios to add JTAG mode)
Info : imx_gpio mmap: pagesize: 4096, regionsize: 131072
Info : clock speed 1000 kHz
Info : SWD DPIDR 0x2ba01477
Info : nrf52.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : Listening on port 3333 for gdb connections
Info : accepting 'telnet' connection on tcp/4444
```

4. Test OpenOCD

Now GDB (port 3333) and the interactive console (port 4444) can be used for
full access to the SoC ARM Cortex-M4 CPU and flash memory.

The following example shows how to read the SoC CPU state:

```
telnet 127.0.0.1 4444
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
Open On-Chip Debugger
> init
> nrf52.cpu curstate
running
```

Type `help` for all available commands.
