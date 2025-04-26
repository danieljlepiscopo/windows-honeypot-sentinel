# Windows Honeypot in Azure with Microsoft Sentinel Attack Map
This project sets up a Windows 10 honeypot in Microsoft Azure, connects it to Microsoft Sentinel, enriches attacker IPs with geolocation data, and visualizes global attacks using an interactive dashboard.

🚀 Project Overview
- Deploy a Windows 10 VM as a honeypot in Azure.
- Collect system and network logs using Sysmon.
- Send logs to Microsoft Sentinel via a Log Analytics Workspace.
- Enrich external IP addresses with geolocation data using Watchlists.
- Build an interactive global attack map inside Sentinel.

🛠️ Setup Guide
1. Create and Configure the Azure VM
_ Deploy a Windows 10 virtual machine.
- Harden it lightly to make it look real but vulnerable (optional: weak passwords, open RDP).
- Install and configure Sysmon for detailed event logging.
Detailed steps: setup/azure_vm_setup.md

2. Set Up Sysmon
- Use a customized sysmon_config.xml to monitor process creation, network connections, and more.
Configuration: setup/sysmon_config.xml

3. Connect VM Logs to Microsoft Sentinel
- Set up a Log Analytics Workspace.
- Connect Windows event logs and Sysmon logs to Sentinel.
- Create a Sysmon Connector if needed.
Instructions: setup/sentinel_connector_setup.md

📊 Visualizing Attacks
Enrichment and Queries
- Upload a Watchlist mapping IPs to country/geolocation info: queries/geolocation_watchlist.csv
- Use custom KQL queries to pull attacker IPs and visualize them: queries/ip_geolocation_query.kql
Dashboard Example:

📂 Project Structure
windows-honeypot-sentinel/
├── README.md
├── architecture-diagram.png
├── setup/
│   ├── azure_vm_setup.md
│   ├── sysmon_config.xml
│   └── sentinel_connector_setup.md
├── queries/
│   ├── geolocation_watchlist.csv
│   ├── ip_geolocation_query.kql
│   └── attack_map_dashboard.kql
├── logs/
│   └── sample_sysmon_events.json
└── images/
    └── dashboard_screenshot.png
    
🧠 Skills Demonstrated
- Cloud security architecture (Azure + Sentinel)
- Windows endpoint security (Sysmon)
- Kusto Query Language (KQL)
- Threat detection and enrichment
- Building security dashboards

📜 License
- MIT License
