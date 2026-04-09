# Azure VM Setup for Honeypot Lab
We're going to create a honeypot using a free subscription on Microsoft Azure. This honeypot is a virtual machine that we're going to intentionally expose to the public internet by removing any firewall. Within minutes or hours, it will begin to be attacked. We will then forward the logs from this VM to a central repository in Azure, where that repo is connected to Microsoft Sentinel (SIEM). We will query the Sentinel to then create an "attack map" of all of the geographical locations where attackers are located.

## Step 1: Create a Free Azure Subscription
- Go to Microsoft's Azure to create an [Azure Free Account](https://azure.microsoft.com/en-us/pricing/purchase-options/azure-account).
- If it doesn't allow you to create a free account, you can create a paid subscription and close it out after you're done with the lab.
- It does ask you to put down a credit card, just to have it on file, but it shouldn't charge you until after the free Azure credits have been used up.
- After your subscription is created, you can log in at [Azure Portal](https://portal.azure.com).

## Step 2: Create the Virtual Machine
This is where we will create our virtual machine, forward our logs to Sentinel, and display the attack map. First, we need to step up a few things.

### Set up supporting resources:

### 1. Create a Resource Group
We need to create a Resource Group, which is almost like a folder for our cloud resources.
- Search "Resource Group" in the search bar.
- Create a new Resource Group.
- Pick the free subscription, call it ``RG_SOC_Lab``, and the region will be ``East US 2``.
  * This will mean that when we create something in the cloud, it will be located somewhere on the East Coast in Microsoft's data centers.
- Then hit "Create." After a minute, our Resource Group should show up.

### 2. Create a Virtual Network
Next, we will need to create a virtual network in order for our VM to eventually be located in (step 3).
- Search "Virtual Network" in the search bar and create a new Virtual Network.
- Pick the free subscription again and click on our resource group name - ``RG_SOC_Lab``.
- For the name of the Virtual Network, we can call it ``Vnet_SOC_Lab``.
- Region will be ``East US 2`` again.
- Keep the default IP/subnet
- Then go to "Review + Create."
- Click "Create" and wait until it's deployed.

### 3. Create the Virtual Machine
Our VM will be the virtual computer in our virtual network; this will be the computer in the cloud!
- Search "Virtual Machine" in the search bar and create the VM.
- Pick the free subscription and click on our resource group name - ``RG_SOC_Lab``.
- For the name of the Virtual Machine, we can call it ``vm-web01-nyc-corp-local``.
    * Try to name it something "official" looking; this is what our honeypot is. Also, make sure you don't call it "honeypot"; we want to deceive our intruders.
- Region will be ``East US 2`` again.
- For Image, pick Windows 10.
- For the Size, you can pick whatever you'd like as long as you cancel within 24 hours. I ended up choosing: ``Standard_B1s`` - 1 vcpu, 1 GiB memory ($7.59/month) (free services eligible).
- Next, create a username and password. This will be associated with the VM.
- Don't forget to check, "I confirm I have an eligible Windows 10/11 license with multi-tenant hosting rights."
- Double-check that your Virtual Network is ``Vnet_SOC_Lab`` and the Subnet is the default (10.0.0.0/24) in the Networking tab.
- Click "Delete Public IP and NIC when VM is deleted".
- Move to the Monitoring tab, and click "Disable" for Boot Diagnostics.
- Move to Review + Create and then Create - You should see a "Validation Passed" once completed.
- Once it's deployed, you'll see the "Your deployment is complete."
If you go back to the Resource Group ``RG_SOC_Lab``, you'll now see several virtual computer components for the VM.

### 4. Edit Network Security Group (NSG)
Up next, we need to edit our VM's security group to allow bad actors to access the VM on our virtual network.
- Click on the Network Security Group option, it should be labeled: ``vm-web01-nyc-corp-local-nsg`` (Based on our VM).
- In the Inbound Security Rules, we will delete the 300 RDP (Remote Desktop Protocol) default rule and create a new inbound rule.
- In Settings > Inbound security rules, we will add an Inbound Security Rule.
- Here are the Inbound Security Rule parameters to add:
  * Make sure Source is set to "**Any**"
  * Source Port Ranges is set to "*"
  * Destination set to "**Any**"
  * Service is set to "**Custom**"
  * Change Destination Port from 8080 to "*"
  * Keep Priority set to "**100**"
  * Change the name to ``DANGER_AllowAnyCustomAnyInbound``

!!! **WARNING**: This will allow ANY inbound traffic to freely try to log in to this VM - this is extremely dangerous, so be careful !!!

### 5. Disable Windows Firewall on the VM
With our NSG setup, we're going to log into our VM and turn off the Windows Firewall on it.
- Search "Virtual Machine" in the search bar and click on our VM - ``vm-web01-nyc-corp-local``.
- Find the Public IP address of the VM. This is what we will use to connect.
- From your own Windows computer, go to the Start menu, and type in "Remote Desktop Connection" (This will allow us to connect to the public IP of the VM).
  * If you're running MacOS, go to the App Store and download "Windows Add". This will allow you to connect to the VM.
- Put the public IP address of the VM, and then the VM username and password when it asks for your credentials.
		* If you accidentally forget your VM login information, you can either delete the VM and recreate it or go to your VM > Reset password, and here you'll be able to reset it.
- Say "no" to all of the privacy settings of the Windows VM and click "Accept". You will then log into the VM.
- Click on the "Start" menu, and search for ``wf.msc``. 
- The Windows Defender Firewall will pop up, so we'll click on "Windows Defender Firewall Properties".
- Go through the tabs, Domain Profile, Private Profile, and Public Profile, and turn off the "Firewall state". Click Apply and then Ok.

Tip: One way to make sure your VM is working is to ping it via your own device. 
	1. On Windows, search "Windows PowerShell" via your Start Menu.
	2. On MacOS, click the magnifying glass icon and type "**Terminal**".
	* From here, type "ping <your VM Public IP address> and hit enter. Make sure you're getting ICMP pings back successfully.

## Step 3: View Raw Logs on the VM
With Windows Defender turned off, let's make sure we capture the logs associated with the login attempts to our honeypot.
- Move back to your VM and search for "**Event Viewer**" via the Start Menu. Please open it.
- This is where a Windows machine will log information associated with what happens on it.
- Go to **Windows Logs** > **Security**, and inside of here is where all of the "security events" take place.
  * Each Event ID is associated with different security events (i.e., Event ID 4625 = failed login attempts).

## Step 4: Create a Log Analytics Workspace (LAW)
Now, for our VM logs to be saved, we need a workspace to save them to - that's where a LAW comes into place!
- We'll need to go back to Azure and search for "**Log Analytics Workspace**".
- Click "Create Log Analytics Workspace".
- Pick the free subscription and click on our resource group name - ``RG_SOC_Lab``.
- We can name the Log Workspace ``LAW-SOC-Lab``.
- Region will be ``East US 2`` again.
- Click on Review + Create.
- Then click on Create to create the log repo.
- Wait until the LAW has been created and deployed.

## Step 5: Set Up Microsoft Sentinel
Next, we will create our SIEM through Microsoft Sentinel and connect our Log Analytics Workspace to it.
- Search "Sentinel" in the search bar.
- Click "Create Microsoft Sentinel".
- Add Sentinel to the Log Analytics Workspace - ``LAW-SOC-Lab``.
- After it's been deployed, you'll have an instance of Sentinel.

## Step 6: Connect VM to Log Analytics Workspace
To make sure that our VM communicates with our LAW, we will need to make changes within our Sentinel.
- On the left-hand side, click on Content management > Content Hub.
- Search for "Windows Security Events" and install it on the right-hand side of the screen.
- Once installed, in the same window where it was located, it should now say "Manage". Click on it.
- Checkmark "**Windows Security Events via AMA**", on the right-hand side of the screen, you should see a button that says "Open connector page". Click it.
- From here, create a **Data Collection Rule** (this rule is for our VM to send its logs to our SIEM).
- We can name this rule anything; I called mine, ``DCR-Windows``.
- Pick the free subscription and click on our resource group name - ``RG_SOC_Lab``.
- Click on "Next: For Resources".
- Expand the Azure Subscription > Resource Group > and click on our VM: ``vm-web01-nyc-corp-local``. Click on "Next: Collect >".
- On the Collect tab, keep the setting to "All Security Events". Click on "Next: Review + create >".
The Azure Monitor Windows Agent will take a while for the extension to become available (could be 10-30 minutes).

## Step 7: Query Logs with KQL
Next, we want to be able to query our logs in order to find relevant information.
- If we go to the Log Analytics workspace and click on our Log Workspace ``LAW-SOC-Lab``.
- From here, go to "Logs", and we can now create database queries to query our logs.
- Typing in "SecurityEvent" and running the table query should result in log data.
- We can type in this KPL query to see failed login attempts with these column types:
```
SecurityEvent
```
 * This will print the entire table, showing all the logs from the VM.
```
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Account, Computer, EventID, Activity, IpAddress
```
 * This will specify failed login attempts while shifting columns to find more relevant information.
  
## Step 8: Upload Geolocation Data
Each of the Log entries doesn't contain much information about geographical location; however, it does give us an IP address. If we had a list of geographical locations in relation to IP addresses, it should give us an idea of where these attackers are trying to break in in the world.
 - Download the ``geoip-summarized.csv``.
 - Go back to Sentinel and click on our Sentinel instance: ``LAW-SOC-Lab``.
 - From there, go to Configuration > Watchlist and create a new Watchlist.
 - Type this exactly for the Name and Alias: "geoip". Click on Next for Source.
 - Search for the ``geoip-summarized.csv`` file in your downloads folder, and upload it.
 - For SearchKey, click on "network". Next, click on "Next: Review + create >".
You'll end up back at the Watchlist page. Wait until the GeoIP watchlist file has been uploaded 100% (It has 50k records, so it will take a while to upload).

## Step 9: Enrich Logs with Geolocation Data
Now that Sentinel has our watchlist uploaded, we have to query our logs appropriately.
- If we go back to Log Analytics workspace again and click on our Log Workspace ``LAW-SOC-Lab``
- From here, go to "Logs", and we'll put this query in:
```
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
| project TimeGenerated, Computer, AttackerIP = IpAddress, cityname, countryname, latitude, longitude
```
Without going into each of the specifics of this query, this query will allow us to take the geographical data and apply it to the log from our VM. 

## Step 10: Create an Attack Map
Lastly, to create our Attack Map, we'll go back to Sentinel and click on our Sentinel instance again: ``LAW-SOC-Lab``.
- From here, we'll go to Threat Management > Workbooks.
- Click on "+ Add a Workbook".
- Click on Edit and delete the pre-populated elements.
- Copy the JSON data from the ``attack_map_dashboard`` file in the "map" file on this repo.
- Click on "+ Add" and create a query.
- Go to "Advanced Editor". Delete everything in the query that is pre-populated and paste the JSON code into it.
- Click on "Done editing", and the Attack Map will be automatically created.
- Save the Workbook; you can name it anything. I called mine: ``Windows VM Attack Map``. Save it to the resource group.
- Click "Save As".

Tip: You can click on "Map Settings" where the JSON code is located and change the values around.
- This makes the map more customizable when it comes to colors, variables, etc. 

## Conclusion
Once the Attack Map is completed, you might have to wait for more attempts by attackers trying to break into the VM for the map to light up more around the world. Having cities and countries related to the Map can teach you a lot about where potential hackers could be located, and that keeping a device unprotected on the internet leaves you vulnerable to immediate threats. 
