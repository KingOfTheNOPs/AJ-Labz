# Installing Security Onion \(SO\)

![](../.gitbook/assets/image%20%2863%29.png)

### Step 1 Import ISO & Deploy VM

Download the ISO and verify the hash [_HERE_](https://github.com/Security-Onion-Solutions/security-onion)_._ Then import the ISO into your datastore under the ISOs folder   
  
Go to Hosts and Clusters and deploy a new VM. 

![](../.gitbook/assets/image%20%28119%29.png)

Follow the same steps in [Install Virtual Machine](https://aj-labz.gitbook.io/aj-labz/building-the-lab/building-the-lab/install-virtual-machine), but there will be a slight deviation in the hardware selection. There will be two network adapters. One for management and one for sniffing. Keep in mind we used Port ID 230 for the SPAN so you'll want to select that as the sniffing interface during install.   


![](../.gitbook/assets/image%20%28118%29.png)



### Step 2: Installing Security Onion

Log in and select the icon: Install Security Onion  
Run through each page of install, we recommend selecting LVM with the new SecurityOnion installation to allow adding partitions for later on. This isn't needed but makes your life A LOT easier.   
Wait for the files to copy and then restart once complete. Your install is complete but now you need to set up Security Onion!

### Step 3: SO-Setup

Time to log back in and select the icon: Setup.   
  
A window should appear, select continue.   
Then select, "Yes, configure /etc/network/interfaces"  
Your management interface should be the port not configured to recieve the SPAN. 

{% hint style="info" %}
If you forget which interface is assigned to recieve the SPAN then run the following commands in the terminal:   
  
`#view the interfaces  
ip a  
#view incoming traffic, if none comes in then its not the SPAN interface  
sudo tcpdump -i <interface>`   
{% endhint %}

DHCP or Static Window should appear next, pick your poison. We like to manage static IPs on pfSense so we left Security Onion with DHCP.  
  
Next, configure the sniffing interface. Select the one that is set to be the SPAN \(should only be one\)  
  
A confirmation window will appear, select Yes!  
  
Network Configuration is now complete, select reboot!  
  
Now that the network is configured, you'll need to configure the rest of the system.   
  
Select the Setup icon once again.   
Enter the password required and select Yes, Continue from the welcome screen.   
  
This time at the network configuration window select Yes, skip network configuration.   
  
Now you'll be asked for Evaluation or Production mode. If you're looking for a quick install and you're a first time user, select Evaluation mode. You'll be prompted to verify the monitoring interface, create credentials for Squil/Snort/Kibana, and then finally asked to proceed with the changes \(yes\). Evaluation mode won't give you the granularity to select what services should be running from the install. Have no fear, you can change that manually at a later time.   
  
We will be selecting Production mode.   
At the next window we selected New since this is the first and only security onion install.   
You will then be propted to create the account for Squil/Snort/Kibana.   
You'll be prompted for Best Practices or Custom setup, We select Custom to view that all can be configured from the start.   
  
The next window is how many days you want to keep alerts from Squil and how many days to keep the Squil database, we left it at 30 and 7.  
  
The next window is what kind of rulesets do you want. Unfortunately we don't have subscriptions to any of the other rulesets which means we will recieve the new rulesets at a later time\(30-60 days later\). Leave the button on Emerging Threats Open.    
  
We will then be prompted for Snort or Suricata, we primarily enjoy the multithreading feature of Suricata over Snort so we will select Suricata. For additional details visit [HERE](https://cybersecurity.att.com/blogs/security-essentials/open-source-intrusion-detection-tools-a-quick-overview).   
  
Next window will ask if you want to enable the Network Sensor. Select Enable. Leave the rest of the windows on default settings as they appear, by going through Production Mode this allows you to gain an idea of what is being installed. Some items we selected No or changed from default are: File extraction and Full Packet Capture \(select No due to Hard Drive limitations\), and Elastic Stack \(installing Splunk instead\). That's it, Security Onion is installed!  
  
  
  
  
  
   
 



