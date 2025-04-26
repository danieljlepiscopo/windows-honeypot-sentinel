# Windows Honeypot in Azure with Microsoft Sentinel Attack Map
This project sets up a Windows 10 honeypot in Microsoft Azure, connects it to Microsoft Sentinel, enriches attacker IPs with geolocation data, and visualizes global attacks using an interactive dashboard.

## Project Overview 🚀 
- Deploy a Windows 10 VM as a honeypot in Azure.
- Send logs to Microsoft Sentinel via a Log Analytics Workspace.
- Enrich external IP addresses with geolocation data using Watchlists.
- Build an interactive global attack map inside Sentinel.

## Setup Guide 🛠️
### Create/Configure the Azure VM and Connect VM Logs to Microsoft Sentinel
- Deploy a Windows 10 virtual machine.
- Harden it lightly to make it look real but vulnerable (optional: weak passwords, open RDP).
- Set up a Log Analytics Workspace.
- Connect Windows event logs to Sentinel.
  
Detailed steps: [setup/azure_vm_sentinel_setup.md](https://github.com/danieljlepiscopo/windows-honeypot-sentinel/blob/main/setup/azure_vm_and_sentinel_setup.md)

## Visualizing Attacks 📊 
Enrichment and Queries
- Upload a Watchlist mapping IPs to country/geolocation info: [queries/geoip-summarized.csv](https://github.com/danieljlepiscopo/windows-honeypot-sentinel/blob/main/queries/geoip-summarized.csv)
- Use custom KQL queries to pull attacker IPs and visualize them: [queries/ip_geolocation_query.kql](https://github.com/danieljlepiscopo/windows-honeypot-sentinel/blob/main/queries/ip_geolocation_query.kql)

## Project Structure 📂
 ```
windows-honeypot-sentinel/
├── README.md
├── architecture-diagram.png
├── setup/
│   ├── azure_vm_sentinel_setup.md
├── queries/
│   ├── geoip-summarized.csv
│   ├── ip_geolocation_query.kql
├── map/
│   └── attack_map_dashboard.json
├── logs/
│   └── sample_sysmon_events.json
└── images/
    └── dashboard_screenshot.png
 ```
    
## Skills Demonstrated 🧠
- Cloud security architecture (Azure + Sentinel)
- Kusto Query Language (KQL)
- Threat detection and enrichment
- Building security dashboards

## License 📜
- MIT License
