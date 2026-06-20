## 1) SSH SPL Query
### 1) Host-Specific SSH Telemetry Overview
```
source="auth.log" host="jack-virtual-machine" sourcetype="AuthLog" ssh2
```
### 2) Field Extraction & Distribution (Source IP)
```
index=_* OR index=* sourcetype=AuthLog ssh2
```

### 3) High-Volume SSH Source IPs
```
index=_* OR index=* sourcetype=AuthLog ssh2 "Failed password" OR "Accepted password"
| stats count by src_ip
| sort - count
| head 10
```

### 4) Compromised Account Detection (Brute Force Success)
```
index=_* OR index=* sourcetype=AuthLog ssh2 "Failed password" OR "Accepted password"
| stats count(eval(searchmatch("Failed password"))) as failed_attempts, count(eval(searchmatch("Accepted password"))) as successful_logins by src_ip
| where failed_attempts > 5 AND successful_logins > 0
```
### 5) Raw Authentication Event Logs (Failed vs. Accepted)
```
index="main" sourcetype="linux_secure" "Failed password" OR "Accepted password"
```
## 2) Suricata Log Analsys SPL Query 
### 1) Top Intrusion Signatures by Severity and Action
```
source="suricata_mock_logs.json" host="jack-virtual-machine" sourcetype="_json"
| stats count by alert.signature, alert.severity, alert.action
| sort - count
```
### 2) Attacker Profiling: Source IPs & Attack Vectors
```
source="suricata_mock_logs.json" host="jack-virtual-machine" sourcetype="_json"
| stats count as total_attacks, values(alert.signature) as attack_types by src_ip, alert.action
| sort - total_attacks
```

### 3) Detailed Intrusion Alert Audit Trail
```
source="suricata_mock_logs.json" host="jack-virtual-machine" sourcetype="_json" alert.action="blocked" OR alert.action="allowed"
| table _time, src_ip, alert.signature, proto, dest_port, alert.action
| sort - _time
```
### 4) Suricata Raw Intrusion Events (JSON)
```
source="suricata_mock_logs.json" host="jack-virtual-machine" sourcetype="_json"
```

## 3) User Account SPL Query
### 1) User Account Lifecycle Monitoring (Creation & Deletion)
```
index="main" sourcetype="linux_secure" useradd OR "userdel"
```





