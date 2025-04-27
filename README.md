# Phoniebox

Here is my phoniebox that I created for my kids.


## Software installation

1. Install version Bulleseye Lite on a sim card
2. Plus the Raspberry pi to the Ethernet
3. Install wifi drivers (not available with the lite version) and unplug Ethernet
4. 
In my setting I needed to configure pin 4 instead of pin 27

## Configure GPIO settings

I did a very basic configuration
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


## Configure pin 27

1. Open a terminal and create a script to apply the pull-up:

```bash
sudo nano /usr/local/bin/force_gpio27_pullup.sh
```


```bash
#!/bin/bash
raspi-gpio set 27 ip pu
```

## References

Thank very much to the original phoniebox and all tutorials and videos specially:
* https://koboldimkopf.wordpress.com/2020/01/10/tutorial-phoniebox/
* https://www.youtube.com/watch?v=9S8yvfvFSNg


