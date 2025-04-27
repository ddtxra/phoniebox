# Phoniebox

This is the [Phoniebox](https://github.com/MiczFlor/RPi-Jukebox-RFID) I created with my kids.

## Table of Contents
* [Raspbian Installation](#raspbian-installation)
* [OnOff SHIM Installation](#onoff-shim-installation)
* [Configuring the OnOff SHIM](#configuring-the-onoff-shim)
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
* **Pin 7** → **Pin 7** on RPi (GPIO4) (default is Pin 7, but I changed it to Pin 13 (GPIO27) and needed to use wires)
* **Pin 9** → **Pin 9** on RPi (Ground)
* **Pin 11** → **Pin 11** on RPi (GPIO17, trigger)

I had an issue with **Pin 7 (GPIO4)**, because on my Raspberry Pi it was always high and there was no way to set it to low. So the low signal was never sent to the OnOff SHIM and the raspberrypi was not powering off after the shutdown. Changing it to **Pin 13 (GPIO27)** allowed me to set the pin to low, notifying the pin 7 on OnOff SHIM to cut power.

- **Pin 11** triggers the shutdown process.
- Once the shutdown is complete, **Pin 7** (connected to GPIO27 in my case, but usually on GPIO4) is pulled low, cutting power to the Raspberry Pi.

---

### Configuring the GPIO27 to pull up (HIGH) on startup

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

There are 2 wasy to to force the pull-up configuration.


#### Add instruction to /boot/config.txt

Adding the following to `/boot/config.txt`:
   ```bash
   gpio=27=ip,pu
   ```
This unfortunately did not work for me

#### Creating a Service for GPIO27 Pull-Up

1. Create a new script to force the pull-up on GPIO27:

Open a terminal and create a script to apply the pull-up:

```bash
sudo nano /usr/local/bin/force_gpio27_pullup.sh
```
Add the following content:

```
#!/bin/bash
raspi-gpio set 27 ip pu
```
Save and exit (press CTRL + X, then Y to confirm).

2. Make the script executable:

```bash
sudo chmod +x /usr/local/bin/force_gpio27_pullup.sh
```

3. Create a systemd service to run the script after boot:

Create the service:

```bash
sudo nano /etc/systemd/system/force-gpio27.service
```

Add the following content:
```bash
[Unit]
Description=Force GPIO27 pull-up
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/force_gpio27_pullup.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
Save and exit (press CTRL + X, then Y to confirm).

4. Enable the service:
```bash
sudo systemctl enable force-gpio27.service
```

5. Reboot the RPi
```bash
sudo reboot
```

6. Check GPIO27 again:
```bash
raspi-gpio get 27
```

It should now show:

```bash
GPIO 27: level=1 fsel=0 func=INPUT
```

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


