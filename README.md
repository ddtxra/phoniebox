# Phoniebox

This is my [Phoniebox](https://github.com/MiczFlor/RPi-Jukebox-RFID) project, created for my kids.

## Software Installation

1. Install the Bullseye Lite version of Raspberry Pi OS on an SD card.
   ![Raspbian OS](assets/raspbian.png)
2. Insert SD card in your raspberryp and connect it to the Ethernet.
3. Install Wi-Fi drivers (not included in the Lite version), configure wifi settings, make sure everything works and then unplug the Ethernet cable.

![OnOff SHIM](assets/onoffshim.jpg)
![RaspberryPI](assets/raspberrypi.png)

```bash
sudo apt install firmware-brcm80211 wpasupplicant wireless-tools
```
```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
````

```bash
country=CH
network={
        ssid="SSID"
        psk="Password"
}
```


4. Note: In my setup, I needed to configure pin 4 instead of pin 27.

## Configure GPIO Settings

I performed a basic GPIO configuration as follows:

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

## Configure Pin 27

To configure pin 27 with a pull-up resistor:

1. Open a terminal and create a script to apply the pull-up configuration:

   ```bash
   sudo nano /usr/local/bin/force_gpio27_pullup.sh
   ```

2. Add the following content to the script:

   ```bash
   #!/bin/bash
   raspi-gpio set 27 ip pu
   ```

3. Save the file and make it executable:

   ```bash
   sudo chmod +x /usr/local/bin/force_gpio27_pullup.sh
   ```

4. Add the script to `/etc/rc.local` to ensure it runs at startup:

   ```bash
   sudo nano /etc/rc.local
   ```

   Add the following line before `exit 0`:

   ```bash
   /usr/local/bin/force_gpio27_pullup.sh
   ```

5. Save and exit. Reboot the Raspberry Pi to apply the changes.

## References

Special thanks to the original Phoniebox project and all the tutorials and videos, especially:
* [Kobold im Kopf Tutorial](https://koboldimkopf.wordpress.com/2020/01/10/tutorial-phoniebox/)
* [YouTube Tutorial](https://www.youtube.com/watch?v=9S8yvfvFSNg)


