# Custom Detection Rules & Signatures

This directory contains the custom logic, signatures, and configurations written for the security tools in this SOC Home Lab. 

Rather than relying purely on default out-of-the-box alerts, these rules were engineered to provide precise detection, reduce false positives via thresholding, and demonstrate automated active response.

### Repository Contents

* snort_local.rules: Contains signature-based detection rules for ICMP sweeps, Nmap XMAS scans, and SSH credential stuffing.
* suricata_local.rules: Demonstrates the difference between passive detection (`alert`) and active prevention (`drop` and `reject`) using NFQUEUE routing.
* wazuh_ossec_custom.xml: Contains the XML configuration blocks applied to the endpoints for Real-Time File Integrity Monitoring (FIM), local log ingestion, and the automated `firewall-drop` Active Response.

### Lab Scenarios Covered by these Rules:
1.  Network Reconnaissance: Detecting and dropping Nmap and Ping sweeps.
2.  Endpoint Compromise: Detecting unauthorized hidden Linux users added to `/etc/passwd`.
3.  Credential Access: Detecting Hydra SSH brute force attacks.
4.  Automated Incident Response: Utilizing Wazuh Active Response to dynamically rewrite target machine firewall rules to block attacking IPs.
