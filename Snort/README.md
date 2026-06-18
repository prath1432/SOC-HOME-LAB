# Snort Setup: Detection & Prevention (IDS/IPS)

## 1. Installation & Basic Configuration
Installed Snort on the Target Machine to monitor inbound/outbound network traffic.

```bash
sudo apt update && sudo apt install snort -y
sudo cp /etc/snort/snort.conf /etc/snort/snort.conf.bak
sudo nano /etc/snort/snort.conf

(During installation, you will be prompted to enter your network interface and CIDR block, e.g., 192.168.1.0/24)



