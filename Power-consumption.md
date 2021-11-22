The following measurements provide reference figures for USB armory power
consumption.

The measurements have been performed using an Agilent U1242B multimeter.

## Linux

| Device             |                 Idle¹ |  Medium CPU+I/O workload |     High CPU+I/O workload |
|:-------------------|----------------------:|-------------------------:|--------------------------:|
| Mk II - 900 MHz    |  80 mA @ 5 V (400 mW) |   180 mA @ 5 V ( 900 mW) |    240 mA @ 5 V (1200 mW) |
| Mk II - 528 MHz    |  70 mA @ 5 V (350 mW) |   140 mA @ 5 V ( 700 mW) |    170 mA @ 5 V ( 850 mW) |
| Mk I  -   1 GHz    | 110 mA @ 5 V (550 mW) |   270 mA @ 5 V (1350 mW) |    280 mA @ 5 V (1400 mW) |
| Mk I  - 800 MHz    | 110 mA @ 5 V (550 mW) |   250 mA @ 5 V (1250 mW) |    260 mA @ 5 V (1300 mW) |

¹ with frequency scaling and without power management (e.g. CPU suspend)

## [TamaGo unikernel](https://github.com/f-secure-foundry/tamago)

| Device             |     Idle² (@ 198 MHz) |   Mixed CPU+I/O workload |     High CPU+I/O workload |
|:-------------------|----------------------:|-------------------------:|--------------------------:|
| Mk II - 900 MHz    | 130 mA @ 5 V (650 mW) |   170 mA @ 5 V ( 850 mW) |    260 mA @ 5 V (1300 mW) |

² without frequency scaling or power management (e.g. CPU suspend)