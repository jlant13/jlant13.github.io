---
title: Lenovo Tiny 10GB Router
date: 2023-12-6 12:00:00 -0800 
categories: [Homelab]
tags: [guide,firewall,hardware,networking]
img_path: /assets/img/posts/tiny-10gb-router
--- 

# Never Buy A New Router Again 

Are you in the market for a new router and not finding one that suits enough/all of your requirements, such as:

    - Compact
    - Quiet / <50db
    - Low power draw / <50w 
    - Powerful enough to handle multiple Wireguard tunnels and/or IPS/IDS
    - Futureproof-able with 10GB ports
    - Won't bankrupt you 
    
    Then let's build one. 

## Lenovo Tiny PCs

If you're not familiar with the Lenovo Tiny PCs, the ServeTheHome forum has a great [Reference Thread](https://forums.servethehome.com/index.php?threads/lenovo-thinkcentre-thinkstation-tiny-project-tinyminimicro-reference-thread.34925/) that dives deep on the differences and upgrades possible.

There's a ton of great small form factor PCs on the market, highlighted by STH's [Project TinyMiniMicro](https://www.servethehome.com/introducing-project-tinyminimicro-home-lab-revolution/). Maybe you even have a one or two setup as home automation docker hosts, or have stacks of 'em setup as Proxmox or Kubernetes clusters. The models chosen here have one aspect that sets them apart from the rest: a PCIe slot. There's quite a few different cards that could utilize this PCIe slot; a graphics card, a 4 port gigabit ethernet card, a Thunderbolt card, and in our case, a 10gb SFP+ card. Now, it is a proprietary PCIe slot that will require a proprietary riser card, but that's alright, they're fairly easy to find. Also, keep in mind that the PCIe slot is x16 sized, it is only x8 electrically, so not all cards will work.

The models that have this PCIe slot are as follows:

    - M720q, M920q, M920x, P330, P340, P350, P360, and M90q Gen 1/2/3 
    

## Parts 

The model that I chose for my build was a M720q with the following specs:

    - i3-8100T
    - 256GB SSD 
    - 8GB RAM
    - 65w power brick

I picked this up on eBay for $115 shipped in August 2023 and have since seen these sell for around $100 for units with similar specs. 

The i3-8100T is more than capable of handling the daily routing duties of a home router along with some Wireguard tunnels. Even during times of Steam sales, I never see the CPU usage crack more than a few percent. I did add in a 2nd 8GB stick of RAM as I had it lying around, but at idle, it only consumes 2GB of RAM. The SSD was swapped out for a NVMe SSD as the PCI card takes up the space of where the SSD caddy was. 

The PCIe riser card required will vary depending on what model you get. For the Tiny5 Generation (M720q, M920q, M920x, P330) use the x16 riser card, part number _01AJ940_. The Tiny6 and Tiny7 Generation (M90q Gen 1/2, P340, P350) use part number _5C50W00877_. The Tiny8 Generation (M90q Gen 3, P360) use part number _5C50W00910_. 

I picked up my riser card from a US based seller off of eBay for $35 shipped. It's a bit more expensive that other listings but it's not shipping from China and the listing includes the riser card, a bag of screws, and the rear baffle bracket. [Here](https://www.ebay.com/itm/285523904761) is a link to the exact item I purchased. The rear baffle bracket included is for a 4 port network card and the mount holes don't line up with the CX312B, but it does make it look cleaner compared to not using a rear bracket. You may be able to find a STL file and 3d print your own bracket or find someone to print one for you. 

There are a few 10GB cards that will fit in these tiny PCs, but the biggest issue is heat. If you want to drop in whatever card you would like and just leave the lid off, you're more than welcome. I won't judge. But in order to keep the lid on and run a 10GB card, the Mellanox ConnectX-3 CX312A or CX312B work well without running significant temps. Also something to make note of is that 10GB Ethernet runs a fair bit hotter than SFP+ ports. 

I picked up a Mellanox CX312B dual SFP+ port for $30 shipped off eBay and a pair of Cisco DAC cables from [10Gtek](https://www.ebay.com/itm/165695736384) for $18. The DAC or SFP transceivers may be different depending on what network card and switch you have, but these Cisco cables worked perfectly for my Mellanox card and Brocade ICX6450 switch.

That's only $200, and you could probably shave a few more dollars off by searching for a bit better of a deal for the Tiny PC or selling off any extra/unused parts like a SSD, SSD caddy, HDMI module, or graphics card if your version came with one.

## Hardware Installation 

Start by removing the thumbscrew that holds the lid on.

![Thumbscrew](/Tiny10GBRouter2.jpg)
_Remove Thumbscrew_


Remove the lid and remove the SSD/caddy, be careful not to tear the SATA ribbon cable.

![SATA Ribbon Cable](/Tiny10GBRouter5.jpg)
_Don't tear me!_


Unscrew the HDMI module, only the black screw needs to be undone as the silver screws mount the module to the rear bracket.

![HDMI module](/Tiny10GBRouter6.jpg)
_Just the black screw_


Remove the two screws holding the rear bracket on.

![Rear Bracket](/Tiny10GBRouter7.jpg)
_2 screws_


Install the PCIe riser card. Wait to tighten down the riser card support until the network card is installed, it will help if you need to wiggle the cards into position if that screw isn't installed yet.

![Riser Card](/Tiny10GBRouter9.jpg)
_Riser card above PCIe slot_


Install new Rear Bracket.

![New Bracket](/Tiny10GBRouter10.jpg)
_New bracket, 2 screws_


Bend the SATA ribbon cable out of the way or remove it. 

![Bend Cable](/Tiny10GBRouter11.jpg)
_Bend cable towards the CPU_


Line up network card against the riser card and make sure to fully seat it.

![Lined Up](/Tiny10GBRouter12.jpg)
_Line up and seat network card_


Flip over TinyPC and slide off the bottom plate to expose the RAM slots and NVMe slots. Install any extra RAM and NVMe SSD. If using a 2280 sized NVMe, remove the spare clip from the shorter position, it may cause some unnecessary stress to have a full sized NVMe flex over the clip.

![Bottom](/Tiny10GBRouter13.jpg)
_Shot of RAM and NVMe slots on bottom of Tiny PC_ 


Replace both the bottom plate and top lid. 

![Rear Final Shot](/Tiny10GBRouter14.jpg)
_Rear shot of fully built Tiny PC_ 


With all the parts installed and the Tiny put back together, it's ready for software.


## Software Installation 


What router operating system you choose to run is up to you, whether is OPNsense, pfSense, Router OS, VyOS, Sophos XG, or anything else, here is where you would do your install. I chose to do a fresh install of OPNsense as it's what i've been using at home for the past couple of years and haven't run into any issues outside of operator error (setting up ACLs and firewall rules before coffee is not recommended). As always the simplest way that i've found to do OS installs is to boot from a 128GB USB running [Ventoy](https://www.ventoy.net/en/index.html) that holds all of my OS ISOs. 

I did skip the initial install process as i'm still in the process of writing a guide for a OPNsense Baseline Configuration and i'll link it here once it's finished. 

To start, the Mellanox card is disabled by default and won't allow it's interfaces to be assigned. That means we only have one working port during the software installation. The process to enable the Mellanox card is fairly simple. After the first reboot post initial installation, plug a ethernet cable into the built-in gigabit port on the Tiny, and plug the other end into your switch. With only one port available, OPNsense should default that port to be the LAN port and assign it an ip address of 192.168.1.1. Set your laptop or PC to and address in the same subnet (eg. 192.168.1.10) and set the gateway to 192.168.1.1. Open your browser to 192.168.1.1 and login using the default credentials (if you didn't change the password the credentials should be installer//opnsense). Go through the config wizard, you can always reset to factory defaults and run through the config wizard again, so most of the defaults here are fine. My quick settings are as follows:

```
    Hostname: OPNsense
    Domain: home
    Primary DNS: 1.1.1.1
    Secondary DNS: 9.9.9.9
    IPv4 Config Type: DHCP
    LAN IP Address: 192.168.1.1/24 
```

Open up the _System_ tab and select _Settings_ then select _Administration_

![Settings Tab](/Tiny10GBRouter18.png)
_System > Settings > Administration_ 


Scroll down to the _Secure Shell_ section and enable SSH with the following default settings. _Please note that these default settings may not be the most secure settings for long term use and you may change these settings once finished enabling the interfaces_

![Secure Shell](/Tiny10GBRouter19.png)
_SSH Settings_


With SSH enabled open up a terminal session and ssh into OPNsense

```bash
    ssh installer@192.168.1.1
    # yes to save ssh key
    # enter password
```

Once logged in and at the main OPNsense menu enter _8_ to open a terminal session on the OPNsense host. Then run the following command to open a bootloader config file: 

```bash 
    vi /boot/loader.conf/local
``` 

With the config file open, enter the following line which enables OPNsense to load the Mellanox driver at boot:

```bash
    # add to top of file
    mlx4en_load="YES"
        
    # i to enter insert mode
    # esc to exit insert mode
    # :w to save 
    # :q to exit 
```

Once the config file has been saved, reboot and ssh back into OPNsense. At the main menu enter _1_ to assign interfaces. Now you should see that you have all of your interfaces available, in my case _em0_, _mlxen0_, and _mlxen1_. If you have a fiber connection running from your upstream/modem, set one of the SFP+ ports as WAN and the rest as LAN ports. I don't have fiber yet, so I have my _em0_ interface as my WAN and both SFP+ ports set as LAN ports. 


## Finished 

At this point you're Tiny router is 10GB enabled and ready to go. Go back and run the config wizard again if you feel or continue your config. Have fun! 

## References
    
- <https://smallformfactor.net/forum/threads/lenovo-m720q-tiny-router-firewall-build-with-aftermarket-4-port-nic.14793/>
- <https://norbertas.com/blog/lenovo-m720q-router/>
- <https://forums.servethehome.com/index.php?threads/lenovo-thinkcentre-thinkstation-tiny-project-tinyminimicro-reference-thread.34925/>
- <https://www.routerperformance.net/opnsense/mellanox-connecx-management-in-opnsense/>