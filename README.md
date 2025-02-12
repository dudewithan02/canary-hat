# Canary v1.0
Canary is a Raspberry Pi HAT that adds a CAN interface over SPI. The full feature list is below:
- CAN network over SPI using MCP2518FD CAN FD controller
- 24V input with 5V 5A output
- RGB interface with 74LVC1G17 buffer
- UART breakout
- I2C header
- 2x CAN outputs (single CAN network) with removeable 120ohm termination resistor jumper for use in both terminal and pass-through modes
- 5V indicator LED
- Full physical access to all 40 GPIO pins (except those used for above functions)

## A note about CAN FD
CAN FD is not currently supported by Klipper, and while Canary uses a CAN FD controller, it will revert to CAN 2.0 in the absence of CAN FD. At the time of writing this (Feb 2025) Klipper only supports CAN 2.0, however if Klipper ever integrates CAN FD into its code, Canary will be hardware-ready.

## Fitting and assembly
It's easy! Place 4 17mm standoffs on your Raspberry Pi, then fit Canary on top, making sure to carefully align the GPIO pins

**For 5V Input** Connect 5V power via the normal USB power port on your Raspberry Pi

**For 24V Input** Connect 24V power via the screw terminal header J2 of Canary

**NOTE** - Do not connect both 5V and 24V power at the same time!

## Installation and Setup of CAN network
For flashing and installation of CAN devices, I highly recommend you read [Esoterical's CANBUS Guide](https://canbus.esoterical.online/). It's THE reference material for CAN networking with Klipper.

There are a couple of extra steps you'll need to follow to set up SPI communication with Canary. For this I will assume you have a working Klipper installation on your Raspberry Pi and you have SSH access.

### Enable SPI and set up the MCP2518FD dtoverlay
1. SSH into your Pi and run `sudo nano /boot/firmware/config.txt'
2. Uncomment `dtparam=spi=on`
3. Scroll to the bottom and add the line `dtoverlay=mcp251xfd,spi0-0,oscillator=40000000,interrupt=25
4. Save and exit
5. Reboot your Raspberry Pi
6. Run `dmesg | grep -i '\(can\|spi\)'` and look for an entry that relates to CAN0 with MCP2518FD being successfully initialized. It will look like the following:
`mcp251xfd spi0.0 can0: MCP2518FD rev0.0 (-RX_INT -PLL -MAB_NO_WARN +CRC_REG +CRC_RX +CRC_TX +ECC -HD o:40.00MHz c:40.00MHz m:20.00MHz rs:17.00MHz es:16.66MHz rf:17.00MHz ef:16.66MHz) successfully initialized.`
7. Go to Esoterical's CANBUS Guide [Getting Started](https://canbus.esoterical.online/Getting_Started.html) and follow the setup guide for TX queue length, initializing the CAN network etc.

**Note** Esoterical's instructions change from time to time, and I'd recommend you visit that page for the most up-to-date guide.

8. Check for a functioning CAN0 network by running `ip -s -d link show can0`. A working and correctly set up CAN0 network will show bitrate 1000000 and qlen 128
9. Continue to flash your CAN toolhead boards as per Esoterical's guide

