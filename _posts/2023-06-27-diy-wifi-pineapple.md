---
title: DIY Wi-Fi Pineapple
date: 2023-06-27 10:00:00 -0800 
categories: [Hacking]
tags: [guide,wifi pineapple,wifi,hacking,hardware]
img_path: /assets/img/posts/diy-wifi-pineapple
--- 

![Pineapples](/brooke-lark-unsplash.jpg)

# The Wi-Fi Pineapple

A Wi-Fi Pineapple is a wireless auditing platform developed by [Hak5](https://hak5.org/) to help network administrators or penetration testers audit network security and identify vulnerabilities in order to strengthen wireless networks against potential attackers. The Wi-Fi Pineapple has been in development since 2008, having many hardware iterations being release over the years along with software updates and an API which enables the use of add-on modules/additional tools and exploits.



Being someone who enjoys building tools, playing around with hardware, and all things DIY, I was curious what it would take to build my own Wi-Fi Pineapple. In my search I came across a really great project on github: [Wi-Fi Pineapple Cloner](https://github.com/xchwarze/wifi-pineapple-cloner)

Rather than extracting the Wi-Fi Pineapple firmware using binwalk, copying the contents to a device-specific version of OpenWRT, and building that version of OpenWrt as developed [here](https://www.securityaddicted.com/2016/11/17/weaponizing-gl-inet-gl-ar150/), which can be unstable and consumes excessive space, xchwarze developed a method that involved patching the original filesystem resulting in a more stable port and is able to work on a wider range of hardware.

## Hardware

There are 211 devices supported by the project. You can see the full list [here](https://github.com/xchwarze/wifi-pineapple-cloner/blob/master/devices.md).

I went with a GL.iNet GL-AR750S-Ext as it had some of the highest specs on the list and I figured that if I wasn't able to get things running or if I realized that a Wi-Fi Pineapple isn't something that I would use a whole lot, I could always reformat and use as intended: a travel router with a VPN back to my home network. I ended up picking one up off of ebay for about $40, used but in great condition. 

## Software

Clean GL-AR750S 3.0 Firmware (OpenWrt 18.06.0) - <https://fw.gl-inet.com/firmware/ar750s/clean/gl-ar750s-3.0-1011_clean.img>

GL-AR750S 3.105 Firmware (OpenWrt 18.06.0) - <https://fw.gl-inet.com/firmware/ar750s/release/openwrt-ar750s-3.105.tar>

OpenWrt v19.07.7 - <https://downloads.openwrt.org/releases/19.07.7/targets/ar71xx/generic/openwrt-19.07.7-ar71xx-generic-gl-ar750s-squashfs-sysupgrade.bin>

Wi-Fi Pineapple Clone firmware - <https://github.com/xchwarze/wifi-pineapple-cloner-builds/blob/main/releases/gl-ar750s-universal-sysupgrade.bin>


## Setup

Once I had received my router, the first step was to flash a specific version of OpenWrt onto it: OpenWrt 19.07.7

The version that was on the router was a much more recent build and rather than just downgrading straight to the version I needed, which could potentially lead to issues such as bricking the device, my plan was to flash an older, clean image and upgrade from there. 

### Flashing OpenWrt 19.07.7 

Start off the with the router powered off  
Connect router to your PC via ethernet using either LAN port
![Router diagram](/router.jpg)
_GL-AR750S Diagram_

Open Settings panel, select Network, and select the _gear_ icon next to your Ethernet connection
![Network Settings](/diy-wifi-pineapple17.png)
_Network Settings_

Manually setup PC to have a static IP of 192.168.1.2, a subnet mask of 255.255.255.0, and a gateway of 192.168.1.1
![Static IP](/diy-wifi-pineapple8.png)
_Static IP_

Boot router into Uboot mode by pressing and holding the Reset button while plugging in power to router, holding the Reset button until the 5GHz LED flashes 5 times

Open browser to <http://192.168.1.1>
- Upload gl-ar750s-3.0-1011_clean.img
![Update Firmware](/diy-wifi-pineapple9.png)
*Update Firmware 3.0*

Flash and wait 5 minutes before rebooting back into Uboot mode

Open browser to <http://192.168.1.1>
- Upload openwrt-ar750s-3.105.tar
![Update Firmware](/diy-wifi-pineapple9.png)
_Update Firmware 3.105_

Flash and wait a few minutes for process to complete

Manually setup PC to have a static IP of 192.168.8.2, a subnet mask of 255.255.255.0, and a gateway of 192.168.8.1
![Static IP](/diy-wifi-pineapple1.png)
_Static IP_

Open browser to <https://192.168.8.1>
Login to verify that firmware upgrade worked, there is no default password, and any password set here will be overwritten in further steps
Under the _Upgrade_ tab, the _Current Version_ should read 3.105

Reboot back into Uboot mode  
Manually setup PC to have a static IP of 192.168.1.2, a subnet mask of 255.255.255.0, and a gateway of 192.168.1.1
![Static IP](/diy-wifi-pineapple8.png)
_Static IP_

Open browser to <http://192.168.1.1>
- Upload openwrt-19.07.7-ar71xx-generic-gl-ar750s-squashfs-sysupgrade.bin
![Update Firmware](/diy-wifi-pineapple10.png)
_Update Firmware 19.07.7_

Flash and wait a few minutes for process to complete

Open browser to <http://192.168.1.1>
- Browser should redirect to <http://192.168.1.1/cgi-bin/luci/>

![Luci Login](/diy-wifi-pineapple4.png)
_No password required to login_

Under the System section on the main page, the _Firmware Version_ should read 19.07.7
![OpenWrt 19.07.7](/diy-wifi-pineapple11.png)
_OpenWrt 19.07.7 Successfully flashed_

### Flash Wi-Fi Pineapple Firmware

With the firmware on the router flashed to the correct version of OpenWrt, now is the time to flash the Wi-Fi Pineapple firmware to the router.

Open terminal  
Change the working directory to Downloads
```bash
cd Downloads
```

copy the Wi-Fi Pineapple firmware to the router
```bash
scp gl-ar750s-universal-sysupgrade.bin root@192.168.1.1:/tmp
```

> I ran into an error here: _Unable to negotiate with 192.168.1.1 port 22: no matching host key type found. Their offer: ssh-rsa_

![SSH error](/diy-wifi-pineapple12.png)
_SSH error_
    
To remedy this, add -HostkeyAlgorithm argument to previous command
```bash
scp -oHostKeyAlgorithms=+ssh-rsa gl-ar750s-universal-sysupgrade.bin root@192.168.1.1:/tmp
```

Once image is uploaded, ssh into router
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.1.1
```

![SSH Success](/diy-wifi-pineapple14.png)
_SSH Successful_

Flash Wi-Fi Pineapple firmware using _sysupgrade_
```bash
sysupgrade -n -F /tmp/gl-ar750s-universal-sysupgrade.bin
```

Wait a few minutes for process to complete

### Configure Wi-Fi Pineapple

Manually setup PC to have a static IP of 172.16.42.2, a subnet mask of 255.255.0.0, and a gateway of 172.16.42.2 
![Static IP](/diy-wifi-pineapple19.png)
_Static IP_

Open browser to Wi-Fi Pineapple panel: <http://172.16.42.1:1471/>

Quickly press Reset/Power button to setup with Wi-Fi disabled  
Press and hold Reset/Power button for 2 seconds to setup with Wi-Fi enabled

> Setup with Wi-Fi _disabled_ is **recommended**.

Continue through Setup configuring as you would like

- Set Root Password
- Management AP
    - SSID:  
    - Pass: 
    - Hide AP: [x]
- Open AP 
    - SSID: 
- Filters Configuration
    - Client Filters: Deny Mode
    - SSID Filters: Deny Mode
    - Firewall Settings:
        - Allow SSH Access via WAN []
        - Allow Web Access via WAN []          
- Save

### Format SD card

Having an SD card will be beneficial as you will be able to install modules and tools to the SD card instead of using up internal storage. It's recommended to format the SD card from the Wi-Fi Pineapple Web UI rather than formatting the SD card on your PC, transferring to the Wi-Fi Pineapple, and immediately installing tools to it. The following steps will show how to format the SD card:

Insert the SD card into the Wi-FI Pineapple

Login and navigate to Advanced

Click the dropdown arrow next to USB & Storage  
![Format SD Card](/diy-wifi-pineapple16.png)
_Format SD Card_

Select Format SD Card  
An alert will showup once format has successfully completed

## Finished

At this point the Wi-Fi Pineapple is complete and ready to run. It may be worth picking up an extra Wi-Fi adapter or two if the hardware you chose doesn't have two built in Wi-Fi adapters. Look for Wi-Fi adapters with the RT5370 ([Panda PAU04](https://www.amazon.com/gp/product/B004AC0L4Y)) or MT7612U ([Alfa ACM](https://store.rokland.com/collections/wi-fi-usb-adapters/products/alfa-awus036acm-802-11ac-dual-band-2-4-5-ghz-wifi-usb-adapter) or [Panda PAU0D](https://www.amazon.com/Panda-Wireless-AC1200-Adapter-Antennas/dp/B0B2QD6RPX)) chipsets. 

## References

- <https://github.com/xchwarze/wifi-pineapple-cloner>
- <https://www.securityaddicted.com/2016/11/17/weaponizing-gl-inet-gl-ar150/>
- <https://docs.hak5.org/wifi-pineapple-6th-gen-nano-tetra/>
- <https://github.com/morrownr/USB-WiFi>