# Installing Security Onion \(SO\)

![](../.gitbook/assets/image%20%2863%29.png)

## Step 1 Import ISO & Deploy VM

Download the ISO and verify the hash [_HERE_](https://github.com/Security-Onion-Solutions/security-onion)_._ Then import the ISO into your datastore under the ISOs folder   
  
Go to Hosts and Clusters and deploy a new VM. 

![](../.gitbook/assets/image%20%28119%29.png)

Follow the same steps in [Install Virtual Machine](https://aj-labz.gitbook.io/aj-labz/building-the-lab/building-the-lab/install-virtual-machine), but there will be a slight deviation in the hardware selection. There will be two network adapters. One for management and one for sniffing. Keep in mind we used Port ID 230 for the SPAN so you'll want to select that as the sniffing interface during install.   
  


![](../.gitbook/assets/image%20%28118%29.png)

\_\_



