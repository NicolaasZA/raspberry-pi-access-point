raspberry-pi-access-point
===================
Guide and config files for deploying a Raspberry Pi Access Point.
Supports the following boards:
- Raspberry PI 3B/3B+/4
- Raspberry PI Zero W/WH

However, this should work for other boards out there. The basic requirements is that the board needs to have:
1. Any Wifi card with:
    1. AP mode support (iw list | grep "Supported interface modes" -A 8)
    2. Driver enabling the OS to use it (obvious one)
2. OS with support for hostapd and bridging.

# Setup Guide
This guide assumes you have a Raspbian installation up and running. Both the desktop and lite versions work since compatibility comes from the kernel/hardware.

## Step 1: Update your Raspbian
```
sudo apt-get update
sudo apt-get upgrade
```

## Step 2: Install hostapd (and stop it for now)
From the Gentoo wiki: Hostapd (Host access point daemon) is a user space software access point capable of turning normal network interface cards into access points and authentication servers. 
TLDR -> hostapd allows a wifi chip to act as an access point.

With this we install bridge-utils, a set of tools we will use later.
```
sudo apt-get install hostapd bridge-utils
sudo systemctl stop hostapd
```

## Step 3: Disable DHCP for wlan0 and eth0
wlan0 is the device name for the Pi's wireless interface, and
eth0 is the device name for the Pi's ethernet interface.
We are disabling DHCP auto-configuration for the two interfaces since we will be configuring them ourselves.
```
sudo nano /etc/dhcpcd.conf
```
And add the following lines at the end:
```
denyinterfaces wlan0
denyinterfaces eth0
```
Save and close using Ctrl+X, Y, Enter

## Step 4: Create the network bridge
A network bridge is a virtual or physical device which connects two different local area networks together. The idea here is to connect the wifi access point to the ethernet interface so we can route traffic from the wifi through the ethernet out to the internet and back.
```
sudo brctl addbr br0
sudo brctl addif br0 eth0
```

### Step 5: Configure interfaces
Here we edit the configurations for the network interfaces, bridge included.
```
sudo nano /etc/network/interfaces
```
Check out the interfaces file provided in this repo for a complete configuration.

### Step 6: Configure hostapd
Next, we will configure our wireless access point, we can do this using a file called hostapd.conf in /etc/hostapd folder.
```
sudo nano /etc/hostapd/hostapd.conf
```
Modify yours to reflect the hostapd.conf in this repository.

You can configure the access point to use different channels, encryptions and modes, but this here is a good default. Be sure to put in your own desired network SSID and password. Check out their [documentation](https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf) for definitions on the settings.

### Step 7: Configure hostapd to use our config file
```
sudo nano /etc/default/hostapd
```
Uncomment this line by removing the #.
```
#DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### Step 8: All done
The Pi is now ready, be sure to reboot then check if everything works.
```
sudo reboot -f
```
