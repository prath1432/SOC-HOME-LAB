# Enterprise SOC Monitoring & Threat Detection Lab

## 📌 Project Overview
This project is a fully functional Security Operations Center (SOC) lab environment. It demonstrates the deployment, configuration, and integration of industry-standard security tools to monitor, detect, and alert on malicious activities. The lab features a dual-host architecture where targeted attacks are generated on one machine and detected/analyzed on a centralized monitoring server.

## 🎯 Objectives
* Deploy and configure a centralized SIEM (Splunk) for log ingestion and analysis.
* Implement Endpoint Detection and Response (Wazuh) for host-level monitoring and File Integrity Monitoring (FIM).
* Set up Network Intrusion Detection Systems (Snort & Suricata) to monitor and alert on malicious network traffic.
* Simulate adversary behaviors (e.g., SSH brute force, privilege escalation, unauthorized file modifications).
* Develop custom detection rules, SPL queries, and operational dashboards for a SOC environment.

## 🏗️ Architecture & Lab Topology
The lab consists of two local Ubuntu hosts operating on the same network segment.

* Machine 1 (SOC Monitoring Server):** Hosts the Splunk Enterprise indexer/search head, Wazuh Manager, and centralized IDS alerts.
* Machine 2 (Target/Victim Machine):** Hosts the Wazuh Agent, Splunk Universal Forwarder, Snort, and Suricata. It acts as the primary log generator and target for simulated attacks.

### Data Flow
 Target Machine (Logs/Events)  ➔  Universal Forwarder / Wazuh Agent  ➔  Splunk / Wazuh Manager (SOC Server) ➔  SOC Analyst (Dashboards & Alerts) 

## 🛠️ Technologies Used
* Operating System: Ubuntu Linux
* SIEM: Splunk Enterprise & Splunk Universal Forwarder
* EDR/XDR: Wazuh Manager & Agent
* NIDS/NIPS: Snort, Suricata
* Protocols/Log Types: Syslog, SSH Auth Logs, Suricata `eve.json`, Snort `alert` logs.

## 💡 Skills Demonstrated
* SIEM deployment and log pipeline engineering.
* Endpoint telemetry collection and FIM configuration.
* IDS/IPS rule creation and network traffic analysis.
* Splunk Search Processing Language (SPL) formulation.
* SOC Dashboard creation and visual metric reporting.
* Incident triage and log analysis.

## 🚀 Installation & Documentation
Detailed, step-by-step documentation for each tool can be found in their respective directories:
1.  [Snort Configuration & Rules](Snort/README.md)
2.  [Suricata Configuration & Alerts](Suricata/README.md)
3.  [Wazuh Deployment & FIM](Wazuh/README.md)
4.  [Splunk Log Ingestion & Dashboards](Splunk/README.md)

## 🔍 Detection Use Cases
* Use Case 1: Detection of New Linux User Creation.
* Use Case 2: File Integrity Monitoring (FIM) for critical system files.
* Use Case 3: SSH Brute Force and Unauthorized Login Attempts.
* Use Case 4: Network Intrusion Detection via Suricata/Snort alerts.


## 📈 Results & Dashboards
The project culminated in a centralized Splunk dashboard providing a unified pane of glass for all security events. 

## 🧠 Lessons Learned
* Log Parsing: Integrating different log formats (like Suricata's JSON output vs. traditional Syslog) requires careful configuration of `inputs.conf` and `props.conf` in Splunk.
* Rule Tuning: Default IDS rules generate significant noise; tuning Snort and Suricata rules is critical for reducing alert fatigue.
* Agent Deployment: Establishing secure communication between endpoints and the centralized server is a foundational SOC engineering skill.

## 👤 Author Information
Name: Prathmesh Salunke
* LinkedIn: https://www.linkedin.com/in/prathmeshsalunke/
* GitHub: https://github.com/prath1432
* Role Target: SOC Analyst L1 
