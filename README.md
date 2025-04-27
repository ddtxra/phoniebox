# Phoniebox

This is the [Phoniebox](https://github.com/MiczFlor/RPi-Jukebox-RFID) I created with my kids.

## Table of Contents
* [Raspbian Installation](#raspbian-installation)
* [OnOff SHIM Installation](#onoff-shim-installation)
* [Configuring the OnOff SHIM](#configuring-the-onoff-shim)
* [Creating a Service for GPIO27 Pull-Up](#creating-a-service-for-gpio27-pull-up)
* [GPIO Button Configuration](#gpio-button-configuration)
* [References](#references)

---

## Raspbian Installation

1. Download and install the Bullseye Lite version of Raspberry Pi OS on an SD card.
   ![Raspbian OS](assets/raspbian.png)
2. Insert the SD card into your Raspberry Pi and connect it to Ethernet.
3. Install Wi-Fi drivers (not included in the Lite version), configure Wi-Fi settings, and verify the connection. Once confirmed, unplug the Ethernet cable.

   Install Wi-Fi drivers:
   ```bash
   sudo apt install firmware-brcm80211 wpasupplicant wireless-tools
   ```

   Configure Wi-Fi settings:
   ```bash
   sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
   ```

   Example configuration:
   ```bash
   country=CH
   network={
           ssid="SSID"
           psk="Password"
   }
   ```

---

## OnOff SHIM Installation

The Phoniebox is powered by a power bank, so I needed a power on/off button to safely cut power to the Raspberry Pi.

This [blog](https://koboldimkopf.wordpress.com/2020/01/10/tutorial-phoniebox/) recommended the [OnOff SHIM from Pimoroni](https://shop.pimoroni.com/products/onoff-shim), which was perfect for my needs.

![RaspberryPI](assets/raspberrypi.png)
![OnOff SHIM](assets/onoffshim.jpg)

### Connecting the GPIO Pins

Connect the OnOff SHIM to the Raspberry Pi GPIO pins as follows:
* **Pin 2** → **Pin 2** on RPi (5V)
* **Pin 6** → **Pin 6** on RPi (Ground)
* **Pin 7** → **Pin 13** on RPi (GPIO27) (default is Pin 7, but I changed it to Pin 13)
* **Pin 9** → **Pin 9** on RPi (Ground)
* **Pin 11** → **Pin 11** on RPi (GPIO17, trigger)

### How It Works

- **Pin 11** triggers the shutdown process.
- Once the shutdown is complete, **Pin 7** (connected to GPIO27) is pulled low, cutting power to the Raspberry Pi.

By default, **Pin 7** on my Raspberry Pi was always high, so the signal was never sent to the OnOff SHIM. Changing it to **Pin 13 (GPIO27)** allowed me to set it to low, notifying the OnOff SHIM to cut power.

---

## Configuring the OnOff SHIM

1. Edit the `cleanshutd.conf` file:
   ```bash
   sudo nano /etc/cleanshutd.conf
   ```

   Update the `poweroff_pin` to match GPIO27:
   ```bash
   # Config for cleanshutd
   trigger_pin=17
   led_pin=17
   poweroff_pin=27  # Changed from default (4) to GPIO27
   hold_time=1
   shutdown_delay=0
   polling_rate=1
   ```

2. Ensure GPIO27 starts as HIGH by adding the following to `/boot/config.txt`:
   ```bash
   gpio=27=ip,pu
   ```

   If this doesn't work, proceed to create a service to force the pull-up configuration.

---

## Creating a Service for GPIO27 Pull-Up

1. Create a script to apply the pull-up configuration:
   ```bash
   sudo nano /usr/local/bin/force_gpio27_pullup.sh
   ```

   Add the following content:
   ```bash
   #!/bin/bash
   raspi-gpio set 27 ip pu
   ```

2. Make the script executable:
   ```bash
   sudo chmod +x /usr/local/bin/force_gpio27_pullup.sh
   ```

3. Add the script to `/etc/rc.local` to run at startup:
   ```bash
   sudo nano /etc/rc.local
   ```

   Add this line before `exit 0`:
   ```bash
   /usr/local/bin/force_gpio27_pullup.sh
   ```

4. Save and exit. Reboot the Raspberry Pi to apply the changes.

---

## GPIO Button Configuration

Below is an example configuration for GPIO buttons:

```
[DEFAULT]
enabled: True
antibouncehack: False

[Random]
enabled: True
Type: Button
Pin: 21
functionCall: functionCallPlayerRandomFolder

[Prev]
enabled: True
Type: Button
Pin: 5
functionCall: functionCallPlayerPrev

[Pause]
enabled: True
Type: Button
Pin: 6
functionCall: functionCallPlayerPause

[Next]
enabled: True
Type: Button
Pin: 13
functionCall: functionCallPlayerNext

[DOWN]
enabled: True
Type: Button
Pin: 19
functionCall: functionCallVolD

[UP]
enabled: True
Type: Button
Pin: 26
functionCall: functionCallVolU
```

---

## References

Special thanks to the original Phoniebox project and all the tutorials and videos, especially:
* [Kobold im Kopf Tutorial](https://koboldimkopf.wordpress.com/2020/01/10/tutorial-phoniebox/)
* [YouTube Tutorial](https://www.youtube.com/watch?v=9S8yvfvFSNg)


