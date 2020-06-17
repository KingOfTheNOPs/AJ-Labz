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
Then select, Yes, configure /etc/network/interfaces  
  
If you forget which interface is assigned to recieve the SPAN then run the following commands in the terminal:   
  
`#view the interfaces  
ip a  
#view incoming traffic, if none comes in then its not the SPAN interface  
sudo tcpdump -i <interface>`   
  
 Select the 





