# RaspberrySpy
Writeup of my project to turn a Raspberry Pi Zero 2 W into a pentesting tool.


## Introduction


## Goals
The goal for this project was to create a tool that can do some cool stuff and that can be used for pentesting. I started by writing out some goals for this project and things I want to be able to do with the Raspberry Pi:

### Requirements Raspberry Pi project:
- Physical attack
  -	I want to be able to set up C&C from the victim device to a remote machine (most likely by posing as a HID). (C&C)
  -	I want to be able to leave the Raspberry Pi in a device and be able to send traffic from the Pi to the victim device. It should (at least theoretically/in very simple cases) allow me to compromise a device remotely after being left in the victim device. (LEAVE)

- Wifi
  -	I want to be able to set up an access point (AP) with a captive portal to show people that I can see what password they entered. The AP needs to be visible from the WiFi settings and automatically spawn a browsers with the captive portal (maybe redirect to Outlook/Gmail or something). (AP)
  -	I also want to be able to deauthenticate selected clients on a WiFi network (bonus: make them connect to my captive portal) (DEAUTH)
  -	I want to be able to sniff and save WiFi traffic (particularly WPA/WPA2 handshakes) (SNIFF)
 
## Current resources
Many tools already exist to do (some of) these things using the Raspberry Pi. Since I did not have a good overview yet on which tool did what, I decided to first review the existing tools to determine if a suitable tool already exists. You can also use this as a list of resources on Raspberry Pi pentesting projects:
| Tool | Number of requirements fulfilled | Requirements fulfilled  |  Link |  Notes |
|---|---|---|---|---|
| Rogueportal  | 1 |  AP  | https://github.com/jerryryle/rogueportal |  Good documentation on how to set up a rogue AP, probably fairly easy to add more functionality from different tools |
| Ethsploiter  | 1 | C&C, LEAVE | https://github.com/maxiwoj/Ethsploiter | Very cool idea, emulates an ethernet card over USB and provides a new network (with the Pi and the victim in it) for the victim. No shell/reverse shell built-in for the Raspberry Pi yet, but this should be easy to add so I'm confident this can satisfy both physical attack requirements  |
| DuckBerry Pi | 2 | C&C, LEAVE | https://github.com/ossiozac/Raspberry-Pi-Zero-Rubber-Ducky-Duckberry-Pi | Since this registers as a HID, I don't know if the C&C requirement can be satisfied |
| SwissArmyPi  | 0  | N/A  | https://github.com/vs4vijay/SwissArmyPi | I think this is not actually a real tool.  |
| pwnhyve  |   |   | https://github.com/whatotter/pwnhyve |   |
| Bettercap |   |   | https://github.com/bettercap/bettercap |   |
| PoisonTap |   |   | https://github.com/samyk/poisontap |   |
|   |   |   |   |   |
