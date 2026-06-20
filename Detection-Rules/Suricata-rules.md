2. The Suricata File
Create this file: Detection-Rules/suricata_local.rules

Plaintext
# ===============================================================================
# SOC HOME LAB: SURICATA CUSTOM IDS/IPS RULES
# Description: High-performance rules demonstrating Alert (IDS), Drop (IPS Stealth), 
#              and Reject (IPS Active Reset) capabilities.
# Author: Jack
# ===============================================================================

# -------------------------------------------------------------------------------
# 1. ICMP (Ping) Reconnaissance
# -------------------------------------------------------------------------------
# IDS Mode: Log the ping but allow traffic.
alert icmp any any -> $HOME_NET any (msg:"IDS: ICMP Ping Detected"; sid:100001; rev:1;)

# IPS Mode (Stealth): Drop the packet silently. Attacker connection times out.
# drop icmp any any -> $HOME_NET any (msg:"IPS: ICMP Ping Dropped"; sid:200001; rev:1;)

# IPS Mode (Active): Reject the packet. Attacker receives "Destination Unreachable".
# reject icmp any any -> $HOME_NET any (msg:"IPS: ICMP Ping Rejected"; sid:300001; rev:1;)

# -------------------------------------------------------------------------------
# 2. SSH Brute Force (Port 22)
# -------------------------------------------------------------------------------
# IDS Mode: Alert on 5 connections within 60 seconds.
alert tcp any any -> $HOME_NET 22 (msg:"IDS: SSH Brute Force Attempt"; flow:to_server; threshold:type both, track by_src, count 5, seconds 60; sid:100002; rev:1;)

# IPS Mode (Stealth): Silently sever the SSH connection if threshold is met.
# drop tcp any any -> $HOME_NET 22 (msg:"IPS: SSH Brute Force Dropped"; flow:to_server; threshold:type both, track by_src, count 5, seconds 60; sid:200002; rev:1;)

# IPS Mode (Active): Send a TCP RST (Reset) packet to tear down the attacker's connection.
# reject tcp any any -> $HOME_NET 22 (msg:"IPS: SSH Brute Force Rejected"; flow:to_server; threshold:type both, track by_src, count 5, seconds 60; sid:300002; rev:1;)

# -------------------------------------------------------------------------------
# 3. Nmap SYN Scan (Stealth Scan)
# -------------------------------------------------------------------------------
# IDS Mode: Detect 20 SYN packets in 10 seconds.
alert tcp any any -> $HOME_NET any (msg:"IDS: Nmap SYN Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 10; sid:100003; rev:1;)

# IPS Mode (Stealth): Drop scanning packets to slow down the attacker's enumeration.
# drop tcp any any -> $HOME_NET any (msg:"IPS: Nmap SYN Scan Dropped"; flags:S; threshold:type both, track by_src, count 20, seconds 10; sid:200003; rev:1;)

# IPS Mode (Active): Reset the connection to actively block the scan.
# reject tcp any any -> $HOME_NET any (msg:"IPS: Nmap SYN Scan Rejected"; flags:S; threshold:type both, track by_src, count 20, seconds 10; sid:300003; rev:1;)
(Note: In a real environment, you uncomment the specific drop or reject rule you want to use depending on your IPS strategy).
