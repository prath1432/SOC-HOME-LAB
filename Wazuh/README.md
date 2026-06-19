## 1. Installation: Manager & Agent
Wazuh operates on a centralized architecture. The Manager sits on your SOC Server to analyze data, and the Agent sits on your Target Machine to forward logs and execute responses.

### A. Install Wazuh Manager (Machine 1 / SOC Server)
Run these commands on your SOC Server. The Wazuh Quickstart script is the industry-standard way to deploy the Manager, Indexer, and Dashboard all at once.

```Bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash ./wazuh-install.sh -a

#(Important: When the script finishes, it will print out the Admin Username and Password in the terminal. Copy these down to log into the web dashboard!)

```
### B. Install Wazuh Agent (Machine 2 / Target Machine)
Run these commands on your Target Machine to install the agent and connect it to your SOC Server. Replace <SOC_SERVER_IP> with the actual IP address of Machine 1.

```
#1. Add the Wazuh repository key and source list
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt update
```

```
#2. Install the agent and point it to the Manager
sudo WAZUH_MANAGER='<SOC_SERVER_IP>' WAZUH_AGENT_NAME='Ubuntu-Target' apt-get install wazuh-agent -y
```
```
#3. Enable and start the agent service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
### C. Verify the Connection
To verify the agent successfully connected to the manager, run this command on your Target Machine:
```
sudo grep -i "Connected to" /var/ossec/logs/ossec.log

#(If successful, it will output a line saying it connected to the manager's IP address).
```


## 2. Detection Use Cases (Endpoint Telemetry)
Wazuh natively parses Ubuntu's /var/log/auth.log and /var/log/syslog to detect malicious endpoint behavior out-of-the-box. We will enhance this by configuring custom monitoring.

### A. File Integrity Monitoring (FIM)
We configure the agent to monitor critical system directories for unauthorized file creations, modifications, or deletions.

Edit the agent's configuration file on the Target Machine:

```Bash
sudo nano /var/ossec/etc/ossec.conf
```
```
#Add the following line under the <syscheck> section to monitor the /etc directory in real-time:

<directories check_all="yes" report_changes="yes" realtime="yes">directory_name</directories>

```
### B. New Linux User Creation
Wazuh automatically decodes authentication logs. When a new user is added to the system via adduser or useradd, Wazuh detects the modification to /etc/passwd and the system log event, triggering Rule 5902 ("New user added to system").

### C. SSH Brute Force Detection
Wazuh monitors SSH authentication attempts. Multiple failed login attempts from the same source IP will trigger Rule 5712 ("SSHD brute force trying to get access to the system").


Restart the Wazuh Manager to apply the changes:

```Bash
sudo systemctl restart wazuh-manager
```

## 3. Execution & Testing (Detailed Attack Simulations)
To validate the EDR capabilities, we will simulate three distinct attack vectors. For the network-based attack (SSH), execute the commands from a separate Attacker Machine. For the host-based attacks (FIM and User Creation), execute them directly on the Target Machine.

### A. Testing File Integrity Monitoring (FIM)
1. The Simulation:
We will simulate an attacker dropping a malicious configuration file or modifying a critical system file in the /etc directory. Run this on the Target Machine:

```Bash
# Create a fake backdoor configuration file
sudo touch /home/jack
```
```
# Modify the file to simulate tampering
sudo bash -c 'echo "bind_shell=true" > /etc/system_backdoor.conf'
```

2. SOC Verification (Wazuh Dashboard):
```
#Log into the Wazuh Web UI (https://<SOC_SERVER_IP>).
#Navigate to Modules ➔ Security events.
#In the search bar, add the filter: rule.id: 550 (or rule.group: syscheck).
#You will see the visual alerts for file creation and modification. Expand the log to see exactly what text was changed.
```

### B. Testing Unauthorized User Creation
1. The Simulation:
Attackers often create local accounts to establish persistence. Run this on the Target Machine to simulate a rogue administrator adding an account:

```Bash
# Add a new user without prompting for interactive details
sudo useradd -m -s /bin/bash rogue_admin
```
2. SOC Verification (Wazuh Dashboard):
```
#Log into the Wazuh Web UI.
#Navigate to Modules ➔ Security events.
#Add the filter: rule.id: 5902 (New user added to system).
#Expand the event to see the decoded Linux system log showing the exact time and host where rogue_admin was created.
```
### C. Testing SSH Brute Force (Detection)
1. The Simulation:
From your external Attacker Machine, launch a credential stuffing attack against the Target Machine. (Note: Ensure you have a wordlist like rockyou.txt or a custom passwords.txt file ready).

```Bash
hydra -l root -P passwords.txt ssh://<TARGET_IP>
```
2. SOC Verification (Wazuh Dashboard):
```
#Log into the Wazuh Web UI.
#Navigate to Modules ➔ Security events.
#Add the filter: rule.id: 5712 (SSHD brute force trying to get access to the system).
#The dashboard will display the alert, highlighting the attacker's Source IP address and the targeted username (root).
```
## 4. Log Analysis & Results
Wazuh normalizes all events into JSON format, which are subsequently ingested by Splunk for our centralized dashboards.

Log Location on Wazuh Manager:
```Bash
sudo tail -f /var/ossec/logs/alerts/alerts.json | jq .
```
