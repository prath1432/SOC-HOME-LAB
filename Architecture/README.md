# Lab Architecture & Network Topology

## IP Addressing Scheme
* SOC Monitoring Server (Machine 1): 192.168.x.10 (Splunk Universal Forwarder, Wazuh Manager, Snort)
* Target Machine (Machine 2): 192.168.x.20 (Suricata, Wazuh Agent, Splunk Enterprise)

## Data Flow Diagram
```text
[ TARGET MACHINE (Ubuntu) ]
       │
       ├── Network Traffic ──> [ Snort & Suricata ] ──> /var/log/suricata/eve.json
       │                                                 /var/log/snort/alerts
       │
       ├── System Events ────> [ Wazuh Agent ] ───────> (Port 1514/1515) ──┐
       │   (FIM, Auth)                                                     │
       │                                                                   │
       └── Log Files ────────> [ Splunk UF ] ─────────> (Port 9997) ───────┤
                                                                           │
                                                                           ▼
                                                             [ SOC MONITORING SERVER ]
                                                             ├── Wazuh Manager
                                                             └── Splunk Enterprise Indexer
                                                                           │
                                                                           ▼
                                                                  [ SOC ANALYST ]
                                                               (Dashboards & Alerts)
