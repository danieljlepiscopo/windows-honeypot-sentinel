# Azure VM Setup for Honeypot Lab

This guide will walk you through setting up a Windows honeypot VM in Azure, forwarding its logs to Microsoft Sentinel, enriching them with geolocation data, and visualizing global attacker IPs on an attack map.

## Step 1: Create a Free Azure Subscription
- Go to [Azure Free Account](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account) to create a free account.
- If a free account isn't available, a paid subscription works — you can close it after the lab.
- A credit card is required for setup, but no charges occur until the free credits are used.
- After setup, log into [Azure Portal](https://portal.azure.com).

## Step 2: Create the Virtual Machine
### Set up supporting resources:

### 1. Create a Resource Group
- Search "Resource Group" > Create new > Name: ``RG_SOC_Lab`` > Region: ``East US 2`` > Create.

### 2. Create a Virtual Network
- Search "Virtual Network" > Create new > Name: ``Vnet_SOC_Lab`` > Attach to ``RG_SOC_Lab`` > Region: ``East US 2`` > Default IP/Subnet > Review + Create > Create.

### 3. Create the Virtual Machine
- Search "Virtual Machine" > Create new > Name: ``vm-web01-nyc-corp-local`` (do not name it "honeypot") > Attach to ``RG_SOC_Lab`` > Region: ``East US 2``.
- Image: Windows 10 > Size: ``Standard_B1s`` (or similar).
- Create a Username/Password.
- Confirm Windows 10 license rights.
- Ensure Networking uses ``Vnet_SOC_Lab``.
- Delete Public IP and NIC when VM is deleted.
- Disable Boot Diagnostics.
- Review + Create > Create.

### 4. Edit Network Security Group (NSG)
- Find the NSG linked to your VM (ex: ``vm-web01-nyc-corp-local-nsg``).
- Delete default RDP inbound rule.
- Add a new inbound rule: Allow **Any** Source, **Any** Port, **Any** Destination, Service: Custom, Priority: **100**, Name: ``DANGER_AllowAnyCustomAnyInbound``.

### 5. Disable Windows Firewall on the VM
- Connect to your VM via Remote Desktop (use the VM’s Public IP address).
- Open ``wf.msc`` > Disable Firewall in Domain, Private, and Public profiles.

## Step 3: View Raw Logs on the VM
- On the VM, open **Event Viewer**.
- Navigate to **Windows Logs** > **Security**.
- Security logs (Event ID 4625 = failed login attempts) are recorded here.

## Step 4: Create a Log Analytics Workspace (LAW)
- Search "Log Analytics Workspace" > Create new > Name: ``LAW-SOC-Lab`` > Attach to ``RG_SOC_Lab`` > Region: ``East US 2`` > Review + Create > Create.

## Step 5: Set Up Microsoft Sentinel
- Search "Sentinel" > Create Microsoft Sentinel.
- Attach Sentinel to your ``LAW-SOC-Lab`` workspace.

## Step 6: Connect VM to Log Analytics Workspace
- In Sentinel > Content Management > Content Hub.
- Search "Windows Security Events" > Install > Manage.
- Select **Windows Security Events via AMA** > Open connector page.
- Create a **Data Collection Rule** (e.g., ``DCR-Windows``).
- Attach it to ``RG_SOC_Lab`` and select your VM ``vm-web01-nyc-corp-local``.
- Choose "All Security Events" > Review + Create.
(Note: The extension may take 10-30 minutes to install.)

## Step 7: Query Logs with KQL
- Go to Log Analytics Workspace > ``LAW-SOC-Lab`` > Logs.
- Run simple KQL queries, such as:
```
SecurityEvent
```
- This will print the entire table, showing all the logs from the VM.
```
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity, IpAddress
```
- This will specify failed login attempts while shifting columns to find more relevant information.
  
## Step 8: Upload Geolocation Data
- Download ``geoip-summarized.csv``.
- In Sentinel > Configuration > Watchlist.
- Create a new Watchlist:
  * Name: ``geoip``
  * Alias: ``geoip``
  * Upload the ``geoip-summarized.csv``.
- SearchKey: ``network``

## Step 9: Enrich Logs with Geolocation Data
- Query to enrich logs:
```
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
| project TimeGenerated, Computer, AttackerIP = IpAddress, cityname, countryname, latitude, longitude
```
## Step 10: Create an Attack Map
- In Sentinel > Threat Management > Workbooks > Add a new Workbook.
- Edit > Delete default elements.
- Add Query > Advanced Editor > Paste the JSON from .....
- Save Workbook (e.g., ``Windows VM Attack Map``).
- Customize the map settings if desired.

✅ You now have a fully functioning Azure-based honeypot that collects security logs, enriches them with geolocation data, and visualizes the attack origins worldwide!
