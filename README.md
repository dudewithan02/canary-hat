# Canary v1.0
Canary is a Raspberry Pi HAT that adds a CAN interface over SPI. The full feature list is below:
- CAN network over SPI using MCP2518FD CAN FD controller
- 24V input with 5V 5A output and polarity protection
- RGB interface with 74LVC1G17 buffer
- UART breakout
- I2C header
- MAX31865 header for PT100/PT1000, with DIP switch selector for 2/3/4-wire configurations
- 2x CAN outputs (single CAN network) with removeable 120ohm termination resistor jumper for use in both terminal and pass-through modes
- 5V indicator LED
- Full physical access to all 40 GPIO pins (except those used for above functions)

![Canary-hat](Images/canary_hat.png)

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

**IMPORTANT** Do not go any further until you run `sudo apt update` and `sudo apt full-upgrade`. I spent way too long diagnosing my own recent setup issues, and all I had to do was update my system to allow the MCP2518FD chip to play nice.

### Enable SPI and set up the MCP2518FD dtoverlay
1. SSH into your Pi and run `sudo nano /boot/firmware/config.txt`
2. Uncomment `dtparam=spi=on`
3. Scroll to the bottom and add the line `dtoverlay=mcp251xfd,spi0-0,oscillator=40000000,interrupt=25`
4. Save and exit
5. Reboot your Raspberry Pi
6. Run `dmesg | grep -i '\(can\|spi\)'` and look for an entry that relates to CAN0 with MCP2518FD being successfully initialized. It will look like the following:

   ![MCP2518FD Initialized](Images/mcp2518fd_initialized.png)


7. Go to Esoterical's CANBUS Guide [Getting Started](https://canbus.esoterical.online/Getting_Started.html) and follow the setup guide for TX queue length, initializing the CAN network etc.

   **Note** Esoterical's instructions change from time to time, and I'd recommend you visit that page for the most up-to-date guide.

8. Check for a functioning CAN0 network by running `ip -s -d link show can0`. A working and correctly set up CAN0 network will show bitrate 1000000 and qlen 128.

   ![Successful CAN0 network setup](Images/can0_network.png)

9. Continue to flash your CAN toolhead boards as per Esoterical's guide



## I2C

To enable these functions, your Raspberry Pi must first be set up as an MCU to be recognised by Klipper. To do this, follow the official [Klipper RPi Microcontroller](https://www.klipper3d.org/RPi_microcontroller.html) documentation.

Once this has been set up, please see the sample canary.cfg for details on how to correctly set up these functions with pin mappings.

## UART KATAPULT INSTRUCTIONS
Before beginning, you need to edit a couple of files on your Raspberry Pi:

```
sudo nano /boot/firmware/cmdline.txt
```
Remove `console=serial0,115200` and save & close.

```
sudo nano /boot/firmware/config.txt
```

Add `dtoverlay=pi3-miniuart-bt` to the end of the file. Save and close.

In your printer.cfg, add the following `[mcu canary]` section:
```
[mcu canary]
serial: /dev/ttyAMA0
restart_method:command
```

Now you are ready to begin flashing Katapult and Klipper

```
sudo apt update
sudo apt upgrade
sudo apt install python3 python3-serial
```

```
test -e ~/katapult && (cd ~/katapult && git pull) || (cd ~ && git clone https://github.com/Arksine/katapult) ; cd ~
```

```
cd ~/katapult
make menuconfig
```

![Katapult build](Images/katapult_build.png)

```
make clean
make
```

```
lsusb
```
![lsusb result](Images/lsusb.png)

```
cd ~/katapult
make flash FLASH_DEVICE=2e8a:0003
```

![Katapult flashing](Images/katapult_flash.png)

```
cd ~/klipper
make menuconfig
```

![Klipper build](Images/klipper_build.png)

```
make clean
make
```

```
sudo service klipper stop
```

```
python3 ~/katapult/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/ttyAMA0
```

```
sudo service klipper start
```

Reboot.

## Updating Klipper

To flash a Klipper update, build the Klipper file using:

```
cd ~/klipper
make menuconfig
```
Select the same options as we did previously:
![Klipper build](Images/klipper_build.png)

Then:
```
make clean
make
```
Stop Klipper running before loading bootloader mode:

```
sudo service klipper stop
```

To enter bootloader mode, run the following:
```
python3 ~/katapult/scripts/flash_can.py -r -d /dev/ttyAMA0
```
You should see the following:

![Bootloader Successful](Images/bootloader.png)

Now finally to flash Klipper to the RP2040, run:
```
python3 ~/katapult/scripts/flash_can.py -f ~/klipper/out/klipper.bin -d /dev/ttyAMA0
```
You should see the following:
![Klipper Update Success](Images/klipper_update.png)

Restart Klipper with the following:

```
sudo service klipper start
```


#### CREDITS

Polar Ted [CAN UART GUIDE](https://github.com/Polar-Ted/RP2040Canboot_Install/tree/main?tab=readme-ov-file)

BTT SKR PICO [UART guide](https://github.com/bigtreetech/SKR-Pico/tree/master/Klipper)

Esoterical [CANBUS Guide](https://canbus.esoterical.online/mainboard_flashing#rp2040-based-boards)
