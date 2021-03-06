### Device tree

Download and compile the example Serial Peripheral Interface (SPI) device tree file [imx53-usbarmory-spi.dts](https://raw.githubusercontent.com/usbarmory/usbarmory/master/software/kernel_conf/mark-one/imx53-usbarmory-spi.dts).

The example device tree enables this SPI configuration on the pin header:

| pin | Mk I         |
|:---:|--------------|
|  1  | Ground       |
|  2  | USB 5V       |
|  3  | SCLK         |
|  4  | MOSI         |
|  5  | MISO         |
|  6  | SS0          |
|  7  | SS1          |


### SPI flash

The example device tree binds a Micron M25P40 SPI flash on SS0 and exposes it
with the Memory Technology Device (MTD) interface. The following kernel logs
confirm a correct SPI configuration:

```
mx51_ecspi_clkdiv: fin: 54000000, fspi: 20000000, post: 0, pre: 2
mx51_ecspi_clkdiv: fin: 54000000, fspi: 20000000, post: 0, pre: 2
m25p80 spi1.0: m25p40 (512 Kbytes)
1 ofpart partitions found on MTD device spi1.0
Creating 1 MTD partitions on "spi1.0":
0x000000000000-0x000000080000 : "test-partition"
```
