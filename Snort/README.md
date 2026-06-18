# Snort Setup: Detection & Prevention (IDS/IPS)

## 1. Installation & Basic Configuration
Installed Snort on the Target Machine to monitor inbound/outbound network traffic.

```bash
sudo apt update && sudo apt install snort -y
sudo cp /etc/snort/snort.conf /etc/snort/snort.conf.bak
sudo nano /etc/snort/snort.conf

#(During installation, you will be prompted to enter your network interface and CIDR block, e.g., 192.168.1.0/24)
```


## 2. Detection vs. Prevention (Custom Rules)
Snort operates in IDS mode (Alerts) by default. To enable IPS mode (Prevention), rule actions are changed from alert to drop. Edit the local rules file:
```
sudo nano /etc/snort/rules/local.rules
```

### A. Ping Scan (ICMP)
Detection Mode:
```
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:100001; rev:1;)
```

Prevention Mode:
```
drop icmp any any -> any any (msg:"ICMP Request Blocked"; sid:100007; rev:1;)
```

### B. SSH Brute Force
Detection Mode:
```
alert tcp any any -> any 22 (msg:"SSH Brute Force Detect"; sid:100003; rev:1;)
```

Prevention Mode:
```
drop tcp any any -> any 22 (msg:"SSH Brute Force Attack Blocked"; sid:100009; rev:1;)
```

### C. Nmap Scan
Detection Mode:
```
alert tcp any any -> any any (msg:"Nmap Scan Detected"; sid:100002; rev:1;)
```
Prevention Mode:
```
drop tcp any any -> any any (msg:"Nmap Scan Blocked"; flags:S; window:1024; threshold:type threshold, track by_src, count 5, seconds 10; sid:100008; rev:1;)
```

## 3. Execution & Testing
Run Snort in IDS Mode (Logging alerts):
```
sudo snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
```
Run Snort in IPS Mode (Inline packet dropping using DAQ):
```
sudo snort -Q --daq afpacket -c /etc/snort/snort.conf -i eth0
```
#(Note: In place of eth0, you must add your actual system interface name. You can check your interface name by using the ifconfig or ip a command in your terminal).

## 4. Attack Simulation (Executing the Threats)

To test your detection and prevention rules, run the following commands from a separate machine (Attacker Machine) targeting your SOC Target Machine. Replace `<TARGET_IP>` with the actual IP address of Machine 2.

### A. Ping Scan (ICMP)
Run a standard ping to test the ICMP rules:
```
ping -c 4 <TARGET_IP>
```

### B. SSH Brute Force (Hydra)
Simulate a brute-force credential attack against the SSH service. (Note: You can use any standard password list like rockyou.txt):
```
hydra -l admin -P passwords.txt ssh://<TARGET_IP>
```

### C. Nmap Scan (XMAS Scan)
Execute an Nmap XMAS scan to trigger the specific Snort/Suricata flags we configured earlier:
```
sudo nmap -sX -p 22,80,443 <TARGET_IP>
```

## 5. Log Analysis & Results
When running as a background service, Snort writes its detections to the default Ubuntu log directory.

### Log Location: /var/log/snort/alert

Command to view live logs:
```
sudo tail -f /var/log/snort/alert
```

> **💡 SOC Analyst Note: Understanding Network Variables**
> * **`$HOME_NET`**: Your defended internal network (e.g., your local IP subnet). 
> * **`$EXTERNAL_NET`**: The untrusted outside world / Internet (Best practice is defining this as `!$HOME_NET`).
> * **`any`**: The universal wildcard (matches every IP address).
>
> **Best Practice:** Always specify traffic direction to reduce false positives and save CPU processing power. Use `$EXTERNAL_NET any -> $HOME_NET <port>` to detect inbound attacks, and `$HOME_NET any -> $EXTERNAL_NET <port>` to detect outbound malware callbacks. Avoid using `any -> any` unless absolutely necessary.

























