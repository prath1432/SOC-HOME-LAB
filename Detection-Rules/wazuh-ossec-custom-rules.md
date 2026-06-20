## 3. The Wazuh File
Wazuh uses a massive ossec.conf file, you should extract just the custom blocks you added and put them in this file so recruiters can read them easily without scrolling through 500 lines of default code.

### A) This block is for File Integrity Monitoring (FIM). It monitors the /etc directory for unauthorized file changes.
```
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <scan_on_start>yes</scan_on_start>
  <directories check_all="yes" report_changes="yes" realtime="yes">/etc</directories>
</syscheck>
```
### B) This block is for SSH Brute Force and New User Detection. It forces Wazuh to read Ubuntu's authentication logs.
```
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
```
### C) This block is for General System Telemetry. It forces Wazuh to read standard operating system events.
```
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>
```
### D) This block is for Active Prevention. When an SSH Brute Force (Rule 5712) is detected, it triggers the local firewall to drop the attacker's IP for 10 minutes (600 seconds). 
```
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>
```
```
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5712,5710</rules_id>
  <timeout>600</timeout>
</active-response>
```
### E) This block is for the Active Response Safelist. It ensures the automated firewall blocking never accidentally locks out your own SOC Analyst IP address.
```
<white_list>127.0.0.1</white_list>
<white_list>^localhost.localdomain$</white_list>
<white_list>192.168.x.x</white_list>
```
