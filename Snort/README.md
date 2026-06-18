# Snort Setup & Configuration Guide

## 1. Installation
Run the following commands on the **Target Machine (Machine 2)**:

```bash
sudo apt update
sudo apt install snort -y

(During installation, you will be prompted to enter your network interface and CIDR block, e.g., 192.168.1.0/24)

## 2. Configuration
Backup the original configuration file before making changes:

Bash
sudo cp /etc/snort/snort.conf /etc/snort/snort.conf.bak
sudo nano /etc/snort/snort.conf
Changes to make in snort.conf:

Find ipvar HOME_NET any and change it to your subnet: ipvar HOME_NET 192.168.x.0/24

Find ipvar EXTERNAL_NET any and change it to !$HOME_NET

