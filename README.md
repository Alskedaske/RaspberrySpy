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
### Enabling SSH & connecting to the Pi

I had already installed Raspbian v12 (Bookworm) on the Pi from a previous project. After signing into the Pi, I enabled SSH using the GUI (under "Preferences):

![image](https://github.com/user-attachments/assets/379beaf8-51f4-4af9-bbc6-6fdf2638dc9f)

(source: https://www.howtogeek.com/768053/how-to-ssh-into-your-raspberry-pi/)

This enables you to connect to the Pi remotely, which is useful since working directly on the Pi can be very slow and it can be cumbersome to have to work using multiple devices.

I also connected the Pi to a WiFi network administered by me and assigned a static IP address to it. You can find a good tutorial on how to do this here: https://raspberrytips.com/set-static-ip-address-raspberry-pi/. 

Please ensure you connect both your Raspberry Pi and your PC to the same network. Once this is done, you can connect to the Pi remotely over SSH using the same credentials you use to sign-in locally on the Pi:
```cmd
ssh username@192.168.x.x
```
`username` should be replaced with your username on the Raspberry Pi and `192.168.x.x` should be replaced with the static IP address you assigned to the Raspberry Pi.


### Installing [pwnhyve](https://github.com/whatotter/pwnhyve)
The wiki guides you through installing pwnhyve: https://github.com/whatotter/pwnhyve/wiki/installing

#### OS
First, we need to get the Kali ARM image for the Raspberry Pi Zero 2 W: https://www.kali.org/docs/arm/raspberry-pi-zero-2-w/

It states that we need to be careful to install the image to the correct drive path, since it will wipe out the microSD card. I first checked my drive paths:
```bash
sudo fdisk -x
```
![image](https://github.com/user-attachments/assets/8e3504ca-cd1e-4410-9c91-6d1656f0b3d1)

This shows the partitions on the device. I also used
```bash
df -h
```
![image](https://github.com/user-attachments/assets/8caadbf2-0cee-4a8d-a437-dad376adb4c5 )

This displays the amount of space available on the file system. Due to the size of the disks, the microSD card appears to be located at `/dev/mmcblk0p2` (size: 29 GB). This is also confirmed up by the Raspberry Pi documentation on partitioning: [https://github.com/raspberrypi/noobs/wiki/Standalone-partitioning-explained](https://github.com/raspberrypi/noobs/wiki/Standalone-partitioning-explained). We will therefore install the image to the drive at `/dev/mmcblk0p2`

I downloaded the Kali ARM image to the Raspberry Pi while in my home directory (`/home/alskedaske`):
```bash
wget https://kali.download/arm-images/kali-2024.2/kali-linux-2024.2-raspberry-pi-zero-2-w-armhf.img.xz
```
This takes ~20 minutes to download.

I tried to install the image on the microSD card but it gave me an error within less than a second. Other commands also give an input/output error. This usually indicates a hardware issue and may be an indication that the installation did not succeed.
![image](https://github.com/user-attachments/assets/328226af-6bae-456b-9217-118182ffeede)

 However, since I was not sure what the Pi was doing, I decided to wait ~1 hour. My thought process was that the installation may be taking place, causing commands other than the core GNU commands (which may be in another parition?) may be failing. I also tried to navigate to the file systems After this, I rebooted the Pi and tried to connect to it again using SSH from my laptop:
 
```cmd
ssh username@192.168.x.x
```

This did not work and after logging into my router, the Pi was not connected to it. I decided to connect a screen and keyboard to it and see what was up. It showed me a CLI with (initramfs):

![IMG20240801173445](https://github.com/user-attachments/assets/dbfc1507-9557-4486-b860-7037c04e2864)


This indicates that the installation of the image did not go as intended. Initramfs is the first root filesystem a machine has access to and upon which other filesystems (e.g. Kali Linux) would be mounted. I took out the microSD card and put it in an SD card reader, which I plugged into my laptop. Now we can see what is stored on the card:

![image](https://github.com/user-attachments/assets/f94df68c-ffbe-4982-9c41-60fd0598a962)
![image](https://github.com/user-attachments/assets/e89ab54b-c13e-41a6-88b5-36e15ea195f6)

As you can see, it shows that the drive has a capacity of only 509 MB. This is pretty much the same as the /dev/mmcblk0p1 partition. This shows that the second partition (/dev/mmcblk0p2) appears to be corrupted. I continued by checking the drive for file system errors and repairing the drive:

![image](https://github.com/user-attachments/assets/b5b9779d-5e90-4101-896c-199dcba93858)

However, since this showed that the drive was fine, I decided to have a closer look with Disk Manager. This showed that the partition status was healthy, but that it did not contain any data. 

<img width="551" alt="image" src="https://github.com/user-attachments/assets/3bee80c7-33d9-4ccd-8693-8fca90ff301a">

I considered formatting the SD card, but since the initial drive is still working, I thought it may be better to first try again to install the Kali ARM image on the SD card. We can always format the SD card if it still does not work.

This time, since I had the SD card plugged into my Windows laptop, I used [balenaEtcher](https://etcher.balena.io/) to flash the ARM image. This took around 10-15 minutes.
![image](https://github.com/user-attachments/assets/f7097297-6974-40f3-bcef-2cd45dcfd1ae)
![image](https://github.com/user-attachments/assets/f70413be-6a66-4ff3-905b-1c66d537cb3f)


After this finished, I unplugged the SD card and plugged it into the Raspberry Pi again. The initial start-up took a while, but now we have a functioning Kali OS on the Raspberry Pi.

Back to following the steps on the `pwhyve` wiki. I already logged into Wifi through the GUI rather than with `nmtui`, but I setup ssh and removed xfce:
```bash
sudo systemctl enable ssh
```
I tested this and it worked. Again, I also decided to assign a static IP address to the Pi to make life easier.,k

Once verified and working, I looked into xfce and lightdm and decided to follow the step on the wiki to delete the files associated with them (this leaves you without a nice GUI desktop environment):
```bash
sudo apt purge xfce4* lightdm*
```

After this, I rebooted the Pi.


#### Kernel stuff
The next part on the Wiki requires us to do some things in the kernel. First of all, we need to be root:
```bash
sudo su
```
Now, we can setup dwc2 and dtoverlay:
```bash
echo dtoverlay=dwc2 | sudo tee -a /boot/config.txt
echo dwc2 | sudo tee -a /etc/modules
echo dtparam=spi=on | sudo tee -a /boot/config.txt
echo "libcomposite" | sudo tee -a /etc/modules
```

I checked this by doing
```bash
tail /etc/modules
```
and
```bash
tail /boot/config.txt
```
![image](https://github.com/user-attachments/assets/74626dad-f914-4327-bd2d-ccb4aed247cb)


Then I cloned the `pwnhyve` repository and moved to it:

```bash
git clone https://github.com/whatotter/pwnhyve && cd pwnhyve
```

I installed the requirements, python3-smbus and bettercap:
```bash
pip install -r requirements.txt
sudo apt-get install python3-smbus bettercap
```
However, I ran into an issue with several of these:
![image](https://github.com/user-attachments/assets/3b7a494a-0b68-42e1-b781-08f768ceed49)

1. E: Package 'python3-smbus' has no installation candidate
I looked into this and there was an [issue](https://github.com/whatotter/pwnhyve/issues/38) opened for someone else with this problem. Apparently this package is not needed yet, so I will skip it for now.

2. E: Unable to locate package bettercap
In the same [issue](https://github.com/whatotter/pwnhyve/issues/38) it is stated that bettercap is a problem, but it is not needed if you do not use the WiFi module, which is broken on the Raspberry Pi 2 W.

This is strange, because `bettercap` should also be included in Kali by default. I tried to look for some solution and managed to download it using `apt`:
```bash
sudo apt update
```
```bash
sudo apt install bettercap --fix-missing
```
![image](https://github.com/user-attachments/assets/6fcc5667-4a71-4785-bdc4-9946f2eeac99)

I used `--fix-missing` because without it, it gave me an error and suggested to use this option, which worked. While the wiki says not to upgrade the pi, I think running `update` may be fine. Nevertheless, since this was the first way I found to get access to `bettercap` (which is necessary to satisfy 3/5 requirements), it is worth a try. If it does not succeed, I can always flash the SD card again.

Next, I needed to setup the USB modules:

```bash
sudo cp ./core/installation/pwnhyveusb /bin/ && sudo chmod +x /bin/pwnhyveusb
mkdir /mnt/otterusb
```
This returned the message that `/mnt/otterusb` already exists, which should not be a problem
![image](https://github.com/user-attachments/assets/8e84f12d-1aa9-4d21-bd13-6571b648810f)

Now we create the USB drive file:
```bash
sudo dd if=/dev/zero of=/piusb.bin bs=65535 count=65535 
mkdosfs /piusb.bin
```


### USB module run on boot

We need to ensure the module runs on boot. The wiki suggests using `rc.local`, which was not present in `/etc` yet, so I created it with `nano`. I did some research in how to use the `rc.local` file and found that you can just create it if it does not yet exist. I added the lines on the wiki and `cat`ed the file to check its contents:
![image](https://github.com/user-attachments/assets/426d83fd-cf0a-4a52-9f7c-09f0e6ae471d)

### Making `pwnhyve` start on boot.
The wiki says to do this:
```bash
cp ./core/installation/startup.sh /bin/
mv /bin/startup.sh /bin/pwnhyveStart
pwd # save the output of this command, in your memory or ctrl+c
chmod +x /bin/pwnhyveStart
nano /bin/pwnhyveStart
```
I did this and replaced all `%cwd%` in the file with pwd value `/home/kali/pwnhyve`:
![image](https://github.com/user-attachments/assets/35b78023-fabc-49ce-8ae0-4ccdb964ea57)

### Final part
I did:
```bash
cp ./core/installation/pwnhyve.service /etc/systemd/system/
systemctl enable pwnhyve.service
```
![image](https://github.com/user-attachments/assets/7454c69a-d0ce-4b84-a546-4b2ec45fee0e)

and started the pwnhyve service:
```bash
systemctl start pwnhyve.service
```

