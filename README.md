 # RaspberrySpy
Writeup of my project to turn a Raspberry Pi Zero 2 W into a pentesting tool.


## Introduction


## Goals
The goal for this project was to create a tool that can do some cool stuff and that can be used for pentesting. I started by writing out some goals for this project and things I want to be able to do with the Raspberry Pi:

### Requirements Raspberry Pi project:
- Physical attack
  -	I want to be able to set up C&C from the victim device to a remote machine (most likely by posing as a HID). (C&C)
  -	I want to be able to leave the Raspberry Pi in a device and be able to send traffic from the Pi to the victim device. It should (at least theoretically/in very simple cases) allow me to compromise a device remotely after being left in the victim device, in case the initial physical attack did not succeed for some reason. (LEAVE)

- Wifi
  -	I want to be able to set up an access point (AP) with a captive portal to show people that I can see what password they entered. The AP needs to be visible from the WiFi settings and automatically spawn a browsers with the captive portal (maybe redirect to Outlook/Gmail or something). (AP)
  -	I also want to be able to deauthenticate clients on a WiFi network (bonus: make them connect to my captive portal) (DEAUTH)
  -	I want to be able to sniff and save WiFi traffic (particularly WPA/WPA2 handshakes) (SNIFF)
 
## Current resources
Many tools already exist to do (some of) these things using the Raspberry Pi. Since I did not have a good overview yet on which tool did what, I decided to first review the existing tools to determine if a suitable tool already exists. You can also use this as a list of resources on Raspberry Pi pentesting projects:
| Tool | Number of requirements fulfilled | Requirements fulfilled  |  Link |  Notes |
|---|---|---|---|---|
| Rogueportal  | 1 |  AP  | https://github.com/jerryryle/rogueportal |  Good documentation on how to set up a rogue AP, probably fairly easy to add more functionality from different tools |
| Ethsploiter  | 1 | C&C, LEAVE | https://github.com/maxiwoj/Ethsploiter | Very cool idea, emulates an ethernet card over USB and provides a new network (with the Pi and the victim in it) for the victim. No shell/reverse shell built-in for the Raspberry Pi yet, but this should be easy to add so I'm confident this can satisfy both physical attack requirements  |
| DuckBerry Pi | 2 | C&C, LEAVE | https://github.com/ossiozac/Raspberry-Pi-Zero-Rubber-Ducky-Duckberry-Pi | Since this registers as a HID, I don't know if the C&C requirement can be satisfied |
| SwissArmyPi  | 0  | N/A  | https://github.com/vs4vijay/SwissArmyPi | I think this is not actually a real tool.  |
| pwnhyve  | 5  | C&C, LEAVE, AP, SNIFF, DEAUTH  | https://github.com/whatotter/pwnhyve | This seems to be what I'm looking for. However, it is currently undergoing a rewrite (31-07-2024) so likely to be confronted with bugs  |
| Bettercap | 3 | AP, DEAUTH, SNIFF | https://github.com/bettercap/bettercap | Only for wireless so no physical attack requirements satisfied  |
| PoisonTap | 1 | C&C | https://github.com/samyk/poisontap | Works even when machine is locked/password protected which is cool. However, this tool seems to work primarily by setting up a new network, forcing certain web traffic and redirecting it through the PoisonTap. This is great but does not satisfy many of the current requirements.   |
| P4wnP1 A.L.O.A. |  3  |  C&C, LEAVE, AP | https://github.com/RoganDawes/P4wnP1_aloa  |  Great tool but does not support all WiFi functionality I want. |
| Pwnagotchi  | 2 | DEAUTH, SNIFF | https://github.com/evilsocket/pwnagotchi/  | This looks like an amazing tool specifically targeting WiFi. If there is a way to combine this with some tool to perform the physical attack requirements, this would be amazing. |

This list is probably not exhaustive but good enough for our current purposes, since `pwnhyve` appears to fulfill all requirements.

## Process
1. Enabling SSH & connecting to the Pi

I had already installed Raspbian v12 (Bookworm) on the Pi from a previous project. After signing into the Pi, I enabled SSH using the GUI (under "Preferences):

![image](https://github.com/user-attachments/assets/379beaf8-51f4-4af9-bbc6-6fdf2638dc9f)

(source: https://www.howtogeek.com/768053/how-to-ssh-into-your-raspberry-pi/)

This enables you to connect to the Pi remotely, which is useful since working directly on the Pi can be very slow and it can be cumbersome to have to work using multiple devices.

I also connected the Pi to a WiFi network administered by me and assigned a static IP address to it. You can find a good tutorial on how to do this here: https://raspberrytips.com/set-static-ip-address-raspberry-pi/. 

Please ensure you connect both your Raspberry Pi and your PC to the same network. Once this is done, you can connect to the Pi remotely over SSH using the same credentials you use to sign-in locally on the Pi:
```
ssh username@192.168.x.x
```
`username` should be replaced with your username on the Raspberry Pi and `192.168.x.x` should be replaced with the static IP address you assigned to the Raspberry Pi.


2. Installing [pwnhyve](https://github.com/whatotter/pwnhyve)
The wiki guides you through installing pwnhyve: https://github.com/whatotter/pwnhyve/wiki/installing

First, we need to get the Kali ARM image for the Raspberry Pi Zero 2 W: https://www.kali.org/docs/arm/raspberry-pi-zero-2-w/




