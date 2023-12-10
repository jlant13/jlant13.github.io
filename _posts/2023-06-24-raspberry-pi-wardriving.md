---
title: Raspberry Pi Wardriving Build
date: 2023-06-24 12:00:00 -0800 
categories: [Hacking]
tags: [guide,raspberry-pi,wifi,hardware,kismet]
img_path: /assets/img/posts/raspberry-pi-wardriving
--- 

![Laptop in car](/thisisengineering-unsplash.jpg)

# Wardriving 

The act of locating and identifying Wi-Fi networks by a person in a moving vehicle, using a laptop, smartphone, or any other wireless-enabled device is known as wardriving. The data gathered during wardriving can be used for a variety of reasons and be used by both attackers and defenders. In the case of beneficial motivations, mapping out existing Wi-Fi infrastructure in a city can be helpful if setting up free public Wi-Fi access or ethical hackers scanning networks trying to find security flaws or vulnerabilities in order to improve overall security. On the other hand, people with malicious intent may search for insecure wireless networks in order to steal banking information or personal data. 

Currently there are no laws that specifically prohibit or allow wardriving, though many countries or localities have laws forbidding unauthorized access of computer networks and protecting personal privacy.

## Hardware 

These are the parts that I used in my build:

- Raspberry Pi 4B 4gb 
- 32gb SD Card
- 64gb USB Flash Drive (optional, used as external/extra storage)
- GlobalSat BU-353-S4 GPS Receiver
- Alfa AWUS036ACHM Wi-Fi Adapter
- Anker 20000mAh Power Bank
- Pelican 1150 Hard Case

I do plan on picking up a small HDMI screen and a folding keyboard as playing around with SDR (Software Defined Radio) is on my to-do/project list.

## Software 
 
- Raspberry Pi Imager (Pi OS Lite 64-bit) - <https://www.raspberrypi.com/software/>
- Kismet (Definitely worth giving the Docs a read) - <https://www.kismetwireless.net/>
- Aircrack-ng - <https://www.aircrack-ng.org/doku.php?id=Main>
- Termius (Used on Android) - <https://www.termius.com/>
    

## Setup

This is a headless build, meaning that there are no attached peripherals like a screen, mouse, or keyboard. All initial setup is done through SSH on my PC, but the build is configured to be operated from my Android phone. If you would like to use an actual monitor, keyboard, and mouse setup, go for it, the instructions shouldn't be much different other than the initial boot.

My daily driver's distro is Ubuntu, so the instructions will reflect that, but it shouldn't be too difficult to apply to your own specific OS.

### Flashing Pi OS to SD Card 


Download and install the Raspberry Pi Imager
```bash
sudo apt install rpi-imager
```

Insert the SD card in your computer and run the RPi Imager
```bash
rpi-imager
```
![Choose OS](/RPiWardriving1.png)
_Raspberry Pi Imager_

Once with RPi Imager opens up, select the 'Choose OS' button, select 'Raspberry Pi OS (Other)' option, then select the 64-bit Lite OS option.
![OS (Other)](/RPiWardriving2.png)
_Raspberry Pi OS (Other)_

![64-bit Lite OS](/RPiWardriving3.png)
_Raspberry Pi OS Lite (64-bit_)

Under Storage select your SD card, make sure that A: you choose the right one and B: that any data on the SD card has been saved some where safe as this process will overwrite any data on the card.


### Advanced Settings

Before starting the flashing process we can enable some system settings which will make this process a bit easier as this is a headless build. Under the advanced settings is where these changes will be made. RPi Imager v1.6 is when the menu was introduced, but was hidden by default. To open the menu press CTRL+SHIFT+X. On newer versions, the menu can be accessed by pressing the 'gear' icon.

![Advanced Settings](/RPiWardriving4.png)
_Advanced Settings_

The options to change are the following: 

    - Hostname          # optional
    - Enable SSH        # required 
    - Set user/pass     # required
    - Configure Wi-Fi   # optional
    - Locale Settings   # optional 

Select Save once finished.

If all done editing settings, select the 'Write' button to begin flashing the image to the SD card. This process will take a few minutes to complete before being able to transfer the SD card to your RPi.
![Settings](/)


## Configure

The next few steps can be done either over SSH or with a screen, mouse, and keyboard. The only real difference between the two should be the way you login. Either way, connect the RPi to your local network, by ethernet or Wi-Fi if configured in the previous step, then plug in the GPS receiver, and Wi-Fi adapter before powering on the device. Wait a couple of minutes for the RPi to boot and be assigned an address from your router or however you have DHCP configured. You will need to know what IP address has been assigned to your RPi, and there's a few ways to find that out. Login to your router and check under active DHCP leases, run a nmap scan on your network, or you can use an app like Fing to run a quick scan. 

Once you have the IP address go ahead and SSH in or login.
```bash
ssh USER@IP_ADDRESS
```

Next is pretty standard for any fresh OS install, install updates.
```bash
sudo apt update && sudo apt upgrade -y
```

Unless prompted to reboot, its optional to do so at this point.

### Configure Wi-Fi Adapter 

When picking out Wi-Fi adapters, it can be pretty useful to know which ones work right out of the box without having to deal with tracking down and installing drivers, but making sure that the adapter can be put into monitor mode is probably the most important aspect. Monitor mode allows the Wi-Fi adapter to listen to all packets that are being sent to other wireless destinations rather than only receive packets intended for itself.

Here is a solid resource on Wi-Fi adapters that work well for this application: <https://github.com/morrownr/USB-WiFi>

The Wi-Fi adapter that I used for this build didn't require any extra drivers to be installed, it just worked right out of the box. 

Getting the Wi-Fi adapter into monitor mode is pretty simple and like most things in Linux, there's a few ways to go about it. Using airmon-ng is the way that I've been doing it. You can install airmon-ng on it's own or install the whole aircrack-ng suite, which includes airmon-ng.

Install aircrack-ng or airmon-ng
```bash
sudo apt install aircrack-ng        # install aircrack-ng suite
-or-
sudo apt install airmon-ng          # install airmon-ng as standalone
```

Show wireless adapters 
```bash
sudo airmon-ng
```
Take note of Driver/Chipset. If unsure which is the correct adapter, unplug the Wi-Fi adapter, run airmon-ng, then plug in the adapter, and run airmon-ng again.

Set Wi-Fi adapter to monitor mode
```bash
sudo airmon-ng start INTERFACE    # Wi-Fi adapter usually ends up being wlan1
```


### Install Kismet

The main program that this build is built around is Kismet. Kismet is an open source sniffer, WIDS, wardriver, and packet capture tool for Wi-Fi, Bluetooth, BTLE, wireless thermometers, airplanes, power meters, Zigbee, and more. It operates almost entirely passively and is largely focused on collecting, collating, and sorting wireless data.

The following adds the repository to your sources so that anytime you run sudo update, it will check if Kismet has any updates that can be pulled down. After the repository has been added, do a quick update and install kismet.
```bash
wget -O - https://www.kismetwireless.net/repos/kismet-release.gpg.key --quiet | gpg --dearmor | sudo tee /usr/share/keyrings/kismet-archive-keyring.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/kismet-archive-keyring.gpg] https://www.kismetwireless.net/repos/apt/release/bullseye bullseye main' | sudo tee /etc/apt/sources.list.d/kismet.list >/dev/null
sudo apt update
sudo apt install kismet
```
Towards the end of the install, Kismet will ask if you want to install with suid-root helpers, select Yes.

Hopefully you won't run into any errors during install, but I ran into a dependency error during the install and I'll show you how I fixed the issue.
```bash
# the error that prevented Kismet to install
kismet-core : Depends: libprotobuf17 but it is not installable
```

Running sudo apt install libprotobuf17 would result in an error saying that the package was either deprecated or could not be found. So I had to hunt it down. Ended up searching through <https://www.debian.org/distrib/packages> and found what I was looking for, grabbed the download link, installed the package, and re-ran the kismet install prompt which completed with out issue. 

If you run into the same issue, here's the commands to download and install the package:
```bash
wget http://ftp.de.debian.org/debian/pool/main/p/protobuf/libprotobuf17_3.6.1.3-2_arm64.deb
sudo apt install ./libprotobuf17_3.6.1.3-2_arm64.deb
```

Once Kismet is installed, add your user to the Kismet group
```bash
sudo usermod -aG kismet USER
```

### Configure GPS

Like the Wi-Fi adapter, the GlobalSat GPS receiver works right out of box and didn't require any specific drivers to get it to work. Also, installing Kismet will install the required gps utilities that are needed such as _gpsd_. You can check to see if _gpsd_ is installed by running the following: 
```bash
gpsd --version
```

If it's not installed, go ahead and install it
```bash
sudo apt install gpsd
```

Once you confirm that _gpsd_ is installed, we want to stop these gps services as they interfere with other gpsd instances that get manually ran.
```bash
sudo systemctl stop gpsd.socket
sudo systemctl disable gpsd.socket
```

Detect which device port the GPS is on
```bash
ls /dev/ttyUSB*
```

/dev/ttyUSB0 should be the only result. Then we can bind the GPS receiver to the port that _gpsd_ runs on.
```bash
sudo gpsd /dev/ttyUSB0 -F /var/run/gpsd.sock 
```

To test if the receiver is functioning simply run _gpsmon_. It may take a few minutes before the data begins to stream in, so just be patient.

Edit the kismet.conf file to set the gps device to run on a local port:
```bash
sudo nano /etc/kismet/kismet.conf
```

uncomment the following line: 
```bash
gps=gpsd:host=local,port=2947
```

### Run Kismet

Technically at this point the RPi is setup enough to just start Kismet and begin scanning for networks. Just run Kismet with the Wi-Fi adapter set in monitor mode:
```bash
kismet -c wlan1mon
```

When finished scanning, Kismet will automatically bundle the scan logs into a _.kismet_ file. These _.kismet_ files can converted to PCAP files for use with programs like Wireshark or tcpdump, KML files which are an XML-based markup language for use with Google Earth, or into a _.wiglecsv_ file in preparation for uploading to Wigle.

## Optional Configurations

Going further I will be showing a few more configurations that I made to my build that are entirely optional, but I found to increase the ease of use.

### Kismet Wardrive Mode

Kismet provides a configuration overlay file which preconfigures Kismet to optimize it for basic wardriving - this turns off most logging and data retention, disables processing most packets as much as possible, and configures the radios for optimized AP detection. All this comes at the cost of normal functionality - in wardrive mode, Kismet won’t track non-access-point Wi-Fi devices, perform most IDS functionality, log data packets, or retain handshakes. What it will do is function much better for the specific goal of mapping access points and Bluetooth devices.

If you want to look at all the options that wardrive mode changes/has, the config file is located at /etc/kismet/kismet_wardrive.conf 

To run Kismet in wardrive mode just launch Kismet with the --override wardrive flag:
```bash
kismet -c wlan1mon --override wardrive
```

### Configure remote access through mobile hotspot

Connecting to the RPi from a smartphone was my ideal vision of this build as it didn't require me to carry around a laptop if I wanted to the RPi on a road trip or a hike or wherever else I might want to scan for networks.

On your phone, search for _mobile hotspot_ under Settings. Setup the hostname and password and if the option is available, Network Name (SSID) Broadcast is turned On. Leave the hotspot turned off for the moment.

With the RPi connected to your network via ethernet, power on the RPi and login. Once logged in, you are going to edit the wpa_supplicant.conf file and add in the network details for your mobile hotspot. Open the config file:
```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Add in your network details and make sure the first and second lines are present as well.
![wpa_supplicant.conf](/RPiWardriving10.png)
_wpa-supplicant.conf_

If you would like to store an encrypted psk, you can run the following: 
```bash
wpa_passphrase "SSID" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf
```

This will prompt you to enter in the password to the selected SSID, generate an encrypted string of the password, before adding both the SSID and encrypted string to the bottom of the config file.

If you would like to multiple networks and assign priority of those networks, you may do that as well. Just set up the config file to resemble the following:
```bash
network={
    ssid="SSID_A"
    psk="PASS_A"
    priority=1  #lower priority, connects if SSID_B isn't available
}
network={
    ssid="SSID_B"
    psk="PASS_B"
    priority=2  #higher priority, connects first
}
```

Once the network details have been added to the config file, check the current ip address and take note of the addresses that wlan0 and wlan1 are connected to before rebooting the RPi or reloading the network configurations with the following:
```bash
wpa_cli -i wlan0 reconfigure
```

Turn on the mobile hotspot on your phone and wait a few minutes to allow the RPi to connect before verifying the connecting by checking the the current ip address details with _ip a_ again noting the wlan0 and wlan1 addresses. The ip address given out by your phone shouldn't change each time the hotspot is turned on and off, but may change if the phone is rebooted.

If you are unable to connect your RPi to your hotspot, double check that the network details match those of your mobile hotspot. You might try rebooting again and waiting 5-10 minutes to see if the connection error works itself out. You could run into an issue if the SSID of your mobile hotspot has a space in it, so try changing the name of your hotspot to one without spaces and make sure to update the wpa_supplicant.conf file. If you didn't have the option to enable Network Name Broadcast under your hotspot settings, but you may need to add the following line to your wpa_supplicant.conf file under the network details of your hotspot:
```bash
scan_ssid=1     # normally used to connect to a hidden network
```

Once you have established a connection, check the current ip address of wlan0 with _ip a_ and open up a SSH client app on your phone. Personally I've been using Termius as I've found that it's pretty simple to use and has a ton of great features. Enter in your ip address, username, and password. Connect and accept/verify the ssh key and you should be good. If you fail to connect but can verify that the RPi is connected to the hotspot, double check that the SSH service is running on your RPi.
```bash
sudo systemctl status ssh.service       # get the current status of the ssh service
sudo systemctl is-enabled ssh.service   # check if ssh is enabled at boot
sudo systemctl enable ssh.service       # enable and start ssh at boot
```

### Mount USB drive for extra storage

Setting up a USB drive as a place to store logs will leave the remaining space on your SD card open to installing more programs or tools or anything else you would like.  

Either format the USB drive on your PC before transferring to the RPi or plug the USB drive into the RPi and format it from the RPi by following these steps:
```bash
lsblk -fp                   # list all disks, USB drive is likely /dev/sda
sudo fdisk /dev/sda         # run fdisk and select the USB drive
d                           # delete partition
n                           # create new partition
p                           # set as primary partition type
1                           # set as partition number 1
# press enter on first and last sector to set as default; single partition for whole drive
w                           # writes changes to disk
sudo mkfs.ext4 /dev/sda1    # format drive as ext4
sudo eject /dev/sda         # eject drive
```

With the USB drive now formatted, you can setup the USB drive to mount automatically:
```bash
# unplug and plug USB drive back into the RPi or reboot
sudo mkdir /mnt/usb0        # create a mount point
sudo mount -t ext4 /dev/sda1 /mnt/usb0/      # mount USB drive to mount point
ls -lt /mnt/usb0            # verify USB mount by listing contents
lsblk -fp                   # list all disks, copy the UUID of USB drive
sudo nano /etc/fstab        # edit the fstab file
# add in the following line with the UUID of your USB drive:
UUID='YOUR_USB_UUID' /mnt/usb0 ext4 defaults,auto,users,rw,nofail 0 0
```

Now we can configure Kismet to save logs to the USB drive instead of saving to the SD card. Start off with making a folder on the USB drive to keep things organized.
```bash
sudo mkdir /mnt/usb0/wardrive
```

Edit the kismet_logging.conf file and specify where to save logs to USB drive:
```bash
sudo nano /etc/kismet/kismet_logging.conf
log_prefix=/mnt/usb0/wardrive
```

### Setup bash aliases

I highly recommend setting up bash aliases as they will allow you to set custom default options for a command or program, help minimize the need to correct typos, and can significantly reduce the amount of typing which is required to run commands or programs. They are especially helpful when trying to control the RPi from a mobile device. 

The syntax for creating an alias is:
```bash
alias NAME="command"
alias NAME="command arg1 arg2"
alias NAME="/path/to/script.pl arg1"
```

To save bash aliases, open up your .bashrc file and add your aliases to the file:
```bash
sudo nano .bashrc
```

The following are aliases that I have saved in my .bashrc file:
```bash
# a quick way to clear the terminal
alias c='clear'

#  update and upgrade 
alias update='sudo apt-get update && sudo apt upgrade -y'

# shutdown now
alias shutdown='sudo shutdown -f now'

# reboot now
alias reboot='sudo shutdown -r now'

# set wlan1 to monitor mode and run Kismet in wardrive mode
alias wardrive='sudo airmon-ng start wlan1 && sudo kismet -c wlan1mon --override wardrive'
```

Once you are finished adding in your own aliases, save and close the file. Then reload the .bashrc file to make sure that the aliases are made available to your terminal by using the following command:
```bash
source ~/.bashrc
```

## Conclusion

That's it, you now have a purpose-built headless Raspberry Pi for your wardriving needs. Have fun!

## References

- <https://wigle.net/index>
- <https://www.busysignal.io/>
- <https://www.promiscata.com/>
- <https://hackaday.io/project/176358-yaawp-yet-another-automated-wardriving-project>
- <https://mikesmodz.wordpress.com/2019/02/13/mobile-wardriving-rig/>