
## SOC LAB: RAW SYSTEM & NETWORK LOGS
## Target OS: Ubuntu Linux


---------------------------------------------------------------------------------
### 1. SSH BRUTE FORCE ATTACK
#### Source File: /var/log/auth.log 
#### Description: Hydra tool generating rapid failed login attempts for "test_user".
---------------------------------------------------------------------------------
```
Jun 16 14:32:10 Ubuntu-Target sshd[10245]: Failed password for test_user from 192.168.1.50 port 54321 ssh2
Jun 16 14:32:10 Ubuntu-Target sshd[10245]: Connection closed by authenticating user test_user 192.168.1.50 port 54321 [preauth]
Jun 16 14:32:11 Ubuntu-Target sshd[10246]: Failed password for test_user from 192.168.1.50 port 54328 ssh2
Jun 16 14:32:11 Ubuntu-Target sshd[10246]: Connection closed by authenticating user test_user 192.168.1.50 port 54328 [preauth]
Jun 16 14:32:12 Ubuntu-Target sshd[10247]: Failed password for test_user from 192.168.1.50 port 54335 ssh2
Jun 16 14:32:12 Ubuntu-Target sshd[10247]: Connection closed by authenticating user test_user 192.168.1.50 port 54335 [preauth]

```
---------------------------------------------------------------------------------
### 2. NEW USER CREATION & DELETION
#### Source File: /var/log/auth.log
#### Description: A local user elevating privileges to create and then delete a hidden rogue account named "test_user".
---------------------------------------------------------------------------------
##### User Creation
```
Jun 17 14:40:01 Ubuntu-Target sudo: jack : TTY=pts/0 ; PWD=/home/jack ; USER=root ; COMMAND=/usr/sbin/useradd -m -s /bin/bash test_user
Jun 17 14:40:01 Ubuntu-Target sudo: pam_unix(sudo:session): session opened for user root(uid=0) by jack(uid=1000)
Jun 17 14:40:01 Ubuntu-Target groupadd[12400]: group added to /etc/group: name=test_user, GID=1002
Jun 17 14:40:01 Ubuntu-Target useradd[12400]: new user: name=test_user, UID=1002, GID=1002, home=/home/test_user, shell=/bin/bash
Jun 17 14:40:02 Ubuntu-Target sudo: pam_unix(sudo:session): session closed for user root
```
##### User Deletion
```
Jun 17 14:45:10 Ubuntu-Target sudo: jack : TTY=pts/0 ; PWD=/home/jack ; USER=root ; COMMAND=/usr/sbin/userdel -r test_user
Jun 17 14:45:10 Ubuntu-Target sudo: pam_unix(sudo:session): session opened for user root(uid=0) by jack(uid=1000)
Jun 17 14:45:10 Ubuntu-Target userdel[12500]: delete user 'test_user'
Jun 17 14:45:10 Ubuntu-Target userdel[12500]: removed group 'test_user' owned by 'test_user'
Jun 17 14:45:11 Ubuntu-Target sudo: pam_unix(sudo:session): session closed for user root
```

---------------------------------------------------------------------------------
### 3. FILE INTEGRITY MONITORING (FIM)
#### Source File: /var/ossec/logs/alerts/alerts.json (Wazuh Format)
#### Description: A critical system file in /etc was modified, triggering a checksum mismatch.
---------------------------------------------------------------------------------
```
{
  "timestamp": "2026-06-18T14:50:00.000+0000",
  "rule": {
    "level": 7,
    "description": "Integrity checksum changed.",
    "id": "550",
    "groups": ["ossec", "syscheck"]
  },
  "agent": {
    "id": "001",
    "name": "Ubuntu-Target"
  },
  "manager": {
    "name": "wazuh-manager"
  },
  "syscheck": {
    "path": "/etc/system_backdoor.conf",
    "size_before": 0,
    "size_after": 16,
    "uname_after": "root",
    "gname_after": "root",
    "mtime_after": "2026-06-18T14:50:00",
    "mtime_before": "2026-06-18T14:49:00",
    "uid_after": "0",
    "gid_after": "0",
    "md5_after": "d41d8cd98f00b204e9800998ecf8427e",
    "sha1_after": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "sha256_after": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "event": "modified"
  }
}
```

---------------------------------------------------------------------------------
### 4. ICMP PING & NMAP SYN SCAN
#### Source File: /var/log/suricata/fast.log
#### Description: Network IDS alerting on active reconnaissance from the Attacker IP.
---------------------------------------------------------------------------------
##### ICMP Ping Sweep detected matching Custom Rule SID: 100001
```
06/18/2026-15:01:45.987654  [**] [1:100001:1] IDS: ICMP Ping Detected [**] [Classification: Misc activity] [Priority: 3] {ICMP} 192.168.1.50:8 -> 192.168.1.10:0
```
##### Nmap SYN Scan detected matching Custom Rule SID: 100003
```
06/18/2026-15:05:10.123456  [**] [1:100003:1] IDS: Nmap SYN Scan Detected [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.50:45678 -> 192.168.1.10:80
06/18/2026-15:05:10.124890  [**] [1:100003:1] IDS: Nmap SYN Scan Detected [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.50:45678 -> 192.168.1.10:443
06/18/2026-15:05:10.125102  [**] [1:100003:1] IDS: Nmap SYN Scan Detected [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.1.50:45678 -> 192.168.1.10:22
```
