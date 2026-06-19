## Suricata Setup: High-Performance IDS/IPS
## 1. Installation & Basic Configuration
Suricata generates highly detailed JSON logs (eve.json), making it the industry standard for SIEM integration.

```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update && sudo apt install suricata jq -y
sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak
sudo nano /etc/suricata/suricata.yaml


#(During configuration, ensure HOME_NET is set to your local subnet, e.g., [192.168.1.0/24], and the af-packet interface matches your system interface).

```


## 2. Detection vs. Prevention (Custom Rules)
Suricata can run in IDS mode (Alerts) or IPS mode (Prevention via NFQUEUE). Edit your local rules file to define how Suricata handles specific threats:
```
sudo nano /etc/suricata/rules/local.rules
```

## A. Ping Scan (ICMP)
### Detection Mode (Logs the event but allows the ping):
```
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:100001; rev:1;)
```
### Prevention Mode - Drop (Silently discards the ping - Attacker sees "Timeout"):
```
drop icmp any any -> any any (msg:"ICMP Request Blocked"; sid:100007; rev:1;)
```
### Prevention Mode - Reject (Actively rejects the ping - Attacker sees "Destination Unreachable"):
```
reject icmp any any -> any any (msg:"Reject: ICMP Ping Denied"; sid:100010; rev:1;)
```

## B. SSH Brute Force 
### Detection Mode:
```

alert tcp any any -> any 22 (msg:"SSH Brute Force Detect"; sid:100003; rev:1;)
```
### Prevention Mode - Drop (Silently blocks connection):
```

drop tcp any any -> any 22 (msg:"SSH Brute Force Attack Blocked"; sid:100009; rev:1;)
```
### Prevention Mode - Reject (Forces the attacker's SSH client to disconnect instantly):
```

reject tcp any any -> any 22 (msg:"Reject: SSH Brute Force Connection Refused"; sid:100012; rev:1;)
```
## C. Nmap Scan 
### Detection Mode:
```

alert tcp any any -> any any (msg:"Nmap Scan Detected"; sid:100002; rev:1;)
```
### Prevention Mode - Drop:
```

drop tcp any any -> any any (msg:"Nmap Scan Blocked"; flags:S; window:1024; threshold:type threshold, track by_src, count 5, seconds 10; sid:100008; rev:1;)
```
### Prevention Mode - Reject:
```

reject tcp any any -> any any (msg:"Reject: Nmap  Scan Terminate"; flags:S; window:1024; sid:100011; rev:1;)
```

## 3. Execution & Testing
Run Suricata in IDS Mode (Listening and Logging):

```Bash
sudo suricata -c /etc/suricata/suricata.yaml -i eth0
#(Note: Replace eth0 with your actual system interface name, verifiable via ip a or ifconfig).
```

### Run Suricata in IPS Mode (Inline packet dropping/rejecting):
To actively stop traffic, you must tell Ubuntu's firewall to route traffic through Suricata's NFQUEUE first.

```Bash
sudo iptables -I INPUT -j NFQUEUE
sudo iptables -I OUTPUT -j NFQUEUE
sudo suricata -c /etc/suricata/suricata.yaml -q 0

#(Warning: Ensure you have console access to the machine. Misconfiguring IPTables or Reject rules can instantly sever your own SSH connection).


> 💡 Note: INPUT vs. OUTPUT
> INPUT: Inbound traffic (e.g., an attacker scanning your machine).
> OUTPUT`: Outbound traffic (e.g., internal malware phoning home).
> Routing both to `NFQUEUE` ensures Suricata can block threats in both directions.

```

## 4. Attack Simulation
To test your rules, run these commands from an external Attacker Machine targeting your Suricata sensor. Replace <TARGET_IP> with Machine 2's IP address.

### A. Ping Scan (ICMP)
```Bash
ping -c 4 <TARGET_IP>
```
### B. SSH Brute Force (Hydra)
```Bash
hydra -l admin -P passwords.txt ssh://<TARGET_IP>
```
### C. Nmap Scan 
```Bash
sudo nmap -sS  <TARGET_IP>
```

## 5. Log Analysis & Results
Suricata outputs highly structured logs, separating basic alerts from rich JSON telemetry.

### Standard Log Location (Fast line-by-line alerts):

```Bash
sudo tail -f /var/log/suricata/fast.log
```
### JSON Log Location (Detailed telemetry for Splunk/SIEM):
```
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert" or .event_type=="drop")'
```
