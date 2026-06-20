## 1. The Snort File
```
# ===============================================================================
# SOC HOME LAB: SNORT CUSTOM DETECTION RULES
# Description: Custom IDS signatures for detecting reconnaissance and access attempts.
# Author: Prathmesh
# ===============================================================================
```

### 1. ICMP (Ping) Reconnaissance Detection
Description: Detects basic ping sweeps used to map the internal network.
```
alert icmp any any -> $HOME_NET any (msg:"IDS: ICMP Ping Detected"; sid:100001; rev:1;)
```

### 2. Nmap Scan Detection
 Description: Detects stealthy XMAS scans by looking for the specific FIN, PSH, and URG flags.
```
alert tcp any any -> $HOME_NET any (msg:"IDS: Nmap Scan Detected";  sid:100002; rev:1;)
```

### 3. SSH Brute Force Detection (Thresholding)
 Description: Detects credential stuffing. Triggers if 5 connections are made within 60 seconds from the same source IP.
```
alert tcp any any -> $HOME_NET 22 (msg:"IDS: SSH Brute Force Attempt"; flow:stateless; detection_filter:track by_src, count 5, seconds 60; sid:100003; rev:1;)
```
