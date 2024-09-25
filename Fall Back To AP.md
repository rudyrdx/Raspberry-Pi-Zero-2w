# Steps to Configure Automatic AP Fallback Mode

This guide outlines the steps to configure your Raspberry Pi to automatically switch to Access Point (AP) mode if the primary Wi-Fi network is unavailable.

## 1. Install Required Packages

First, install the necessary packages for access point mode:

```bash
sudo apt-get update
sudo apt-get install hostapd dnsmasq
```

Then, stop the services for now:

```bash
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

## 2. Configure dnsmasq for DHCP

dnsmasq will provide IP addresses (DHCP) when the Raspberry Pi is in AP mode.

Back up the default configuration:

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Create a new configuration file:

```bash
sudo nano /etc/dnsmasq.conf
```

Add the following content:

```bash
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

This configures dnsmasq to provide IP addresses from 192.168.4.2 to 192.168.4.20 on the `wlan0` interface in AP mode.

## 3. Configure hostapd for Access Point

Configure the hostapd service to create the access point.

Create or modify the configuration file:

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Add the following configuration (customize SSID and passphrase):

```bash
interface=wlan0
driver=nl80211
ssid=Pi_AP
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=raspberry
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Edit the hostapd default configuration file:

```bash
sudo nano /etc/default/hostapd
```

Find the line:

```bash
#DAEMON_CONF=""
```

And replace it with:

```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## 4. Configure Static IP for AP Mode

Assign a static IP address to `wlan0` in AP mode.

Edit the `dhcpcd.conf` file:

```bash
sudo nano /etc/dhcpcd.conf
```

Add this at the end:

```bash
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

## 5. Create a Script for Auto-Switching to AP Mode

Create a script to check Wi-Fi network availability and switch to AP mode if needed.

Create the script file:

```bash
sudo nano /usr/local/bin/wifi-check.sh
```

Add the following content:

```bash
#!/bin/bash

# Check if the Pi is connected to a Wi-Fi network
if iwgetid -r; then
    echo "Wi-Fi connected."
    sudo systemctl stop hostapd
    sudo systemctl stop dnsmasq
    sudo systemctl start wpa_supplicant
else
    echo "Wi-Fi not connected. Starting Access Point..."
    sudo systemctl stop wpa_supplicant
    sudo systemctl start dnsmasq
    sudo systemctl start hostapd
fi
```

Save and make it executable:

```bash
sudo chmod +x /usr/local/bin/wifi-check.sh
```

## 6. Run the Script at Boot

Add the script to cron to run at boot.

Open the cron configuration file:

```bash
sudo crontab -e
```

Add the following line:

```bash
@reboot /usr/local/bin/wifi-check.sh
```

## 7. Reboot and Test

Reboot your Raspberry Pi:

```bash
sudo reboot
```

After rebooting:

- If the Pi detects the Wi-Fi network defined in `wpa_supplicant.conf`, it will connect.
- If the Wi-Fi network is unavailable, it will automatically start in AP mode with the configured SSID and password. 
