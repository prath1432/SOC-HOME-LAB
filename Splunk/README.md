## 1. Installation & Basic Configuration
Splunk acts as the central Security Information and Event Management (SIEM) platform for our SOC home lab, indexing logs from Snort, Suricata, and system authentication files for real-time analysis.

## 1.1 Splunk Enterprise Setup

### A. Install Splunk Enterprise (Machine 1 / SOC Server)
Download and install the Splunk Enterprise Debian package on your dedicated SOC server machine.

```Bash
#Install the Splunk .deb package (replace with your exact filename)
sudo dpkg -i splunk-x.x.x-amd64.deb
```
```
#Enable Splunk to start at boot and accept the license agreement
sudo /opt/splunk/bin/splunk enable boot-start --accept-license
```
```
#Start the Splunk service
sudo systemctl start splunk

#(During the first start, you will be prompted to create an administrator username and password. Keep these credentials safe to log into the web interface).
```
### B. Access the Web Interface
```
Open your browser and navigate to http://<SOC_SERVER_IP>:8000.
```
Log in using the admin credentials created during installation.


## 1.2 Splunk Universal Forwarder Setup (Target Machine)
While Splunk Enterprise sits on the SOC Server to analyze data, the Universal Forwarder (UF) acts as a lightweight agent installed on your Target Machine to securely collect and send logs to the indexer.

### A. Install the Universal Forwarder
Download the Universal Forwarder .deb package to your Target Machine (Machine 2) and install it.

```Bash
#Install the downloaded Splunk Forwarder package
sudo dpkg -i splunkforwarder-x.x.x-amd64.deb
```

Start the forwarder service, accept the license agreement, and set it to start automatically on boot:

```Bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
sudo /opt/splunkforwarder/bin/splunk enable boot-start

#(You will be prompted to create an administrator username and password specifically for this forwarder agent).
```
### B. Connect the Forwarder to the SOC Server
You must tell the forwarder exactly where to send its logs. Replace <SOC_SERVER_IP> with the IP address of Machine 1.

```Bash
#Point the forwarder to the Splunk Enterprise receiving port (9997)
sudo /opt/splunkforwarder/bin/splunk add forward-server <SOC_SERVER_IP>:9997 -auth <your_forwarder_username>:<your_forwarder_password>
```
### C. Configure Log Ingestion (Inputs)
Now, tell the forwarder which local files to monitor and send to the SOC server. We will ingest the authentication logs, Suricata JSON alerts, and Snort alerts.

```Bash
#1. Ingest Linux Authentication Logs (SSH/Sudo activity)
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure -index main
```
```
#2. Ingest Suricata JSON Logs
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/suricata/eve.json -sourcetype suricata -index main
```
```
#3. Ingest Snort Alerts
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/snort/alert -sourcetype snort_alert -index main
``` 
### D. Restart & Verify
Restart the Universal Forwarder to apply the new monitoring inputs:

```Bash
sudo /opt/splunkforwarder/bin/splunk restart
```
Verification: You can verify the forwarder is actively connected to the SOC Server by running this command on the Target Machine:
```
Bash
sudo /opt/splunkforwarder/bin/splunk list forward-server

#(If successful, your SOC Server's IP address will be listed under "Active forwards").
```

## 1.3 Enable the Receiving Port (SOC Server)
By default, Splunk does not listen for incoming logs. You must explicitly open a port (usually 9997) on your Splunk Enterprise instance to receive data from your Universal Forwarders.

Run this command on your SOC Server (Machine 1):
```
Bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth <your_admin_username>:<your_admin_password>

#(Note: Replace the username and password with the Splunk Web admin credentials you created during the installation).
```
Alternative Web UI Method:
```
#If you prefer doing this via the dashboard:
#Log into Splunk Web (http://<SOC_SERVER_IP>:8000).
#Navigate to Settings ➔ Forwarding and receiving.
#Under "Receive data", click Configure receiving.
#Click New Receiving Port, enter 9997, and save.
```

## 2. Data Ingestion (Configuring Log Inputs)
To monitor endpoint and network activity, Splunk must ingest log data. This is achieved by configuring the Splunk Web UI on the SOC Server to monitor critical local log files directly, or by ingesting data securely forwarded from our target endpoints.

Verifying File Inputs via Web UI
To verify or manually add data inputs, navigate through the Splunk console using these steps:
```
#Go to Settings ➔ Data inputs.
#Click on Files & directories.
#Click the New Local File & Directory button and configure the following paths:
```

## 3. Threat Detection via SPL (Splunk Processing Language)
Once logs are indexed, a SOC Analyst uses SPL to hunt for threats and build automated detection alerts. Here are the precise queries matching our lab scenarios:

### A. Detecting SSH Brute Force
This query monitors authentication logs for a high volume of failed login attempts from a single source IP.

```
index="main" sourcetype=linux_secure "failed password" 
| stats count by src_ip, user 
| where count > 5
```
### B. Detecting New User Creation
This query tracks whenever an account creation tool is executed or a new user structure is committed to the local system database.
```
index="main "sourcetype=linux_secure ("useradd" OR "adduser" OR "new user")
| table _time, host, user, script, message
```

## 4. Execution, Testing & Dashboard Verification
To prove our SIEM is working correctly, we run attack simulations and verify how they present inside the Splunk console.

### A. Simulating the Telemetry Generation
Run your attack scripts from your external attacker machine (e.g., launching an SSH brute force using Hydra or running an Nmap scan against the target).

```Bash
# Attacker Action Example
hydra -l root -P passwords.txt ssh://<TARGET_IP>
```
### B. Verifying Raw Events in Splunk
```
#Open the Splunk Web UI and go to the Search & Reporting app.
#Set the time picker to Presets ➔ Real-time (1 minute window) or Last 15 minutes.

```
Run the following search to verify logs are updating in real time:
```
index=main sourcetype=linux_secure
```
## 5. Building the L1 SOC Analyst Dashboard
Visualizing events allows a SOC Analyst to spot trends and coordinate incident response quickly.
```
[Attacker Machine] ---> [Target Sensors: Snort/Wazuh] ---> [Splunk Indexer] ---> [SOC Analyst Dashboard Visuals]
```
```
Step-by-Step Dashboard Creation:
1) Run any of the SPL detection queries from Section 3 (e.g., the SSH Brute Force query).

2) Click the Visualization tab below the search bar and select Pie Chart or Bar Chart.

3) Click Save As in the top right corner and select New Dashboard.

4) Configure the dashboard settings:
   - Dashboard Title: L1 SOC Monitoring Dashboard
   - Permissions: Shared in App

5) Click Save to Dashboard.

6) Repeat this process for your User Creation and Network Scan queries, selecting Add to Existing Dashboard to build a single, comprehensive pane of glass.
```
