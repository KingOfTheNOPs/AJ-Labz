# Building the Windows Domain

![](<../.gitbook/assets/image (32).png>)

## Building a Windows Domain

### Step 1: Download and install Windows 2016 server&#x20;

Evaluation ISOs can be downloaded at [here. ](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016?filetype=ISO)They're good for 180 days.\


#### Step 1a: Assign to Port Group

You should already have a Port Group called Servers or something along those lines. Add the PG\_Servers on VCSA.&#x20;

#### Step 1b. Assign Resources

Generally the minimum resources needed are:\
**Processor**: 1 Core\
**RAM**: 512 MB\
**Disk Space**: 32 GB\
\
Screenshot posted below of what I used during install to speed up the process. (resources lowered after install)\


![DC Resources during Install](<../.gitbook/assets/image (22).png>)

### Step 2: Firewall&#x20;

#### Step 2a: Firewall Rules

To start we allowed the DC to talk to the internet (50,80,443) and created an any any rule between the server and workstations. \
\
Restrict the DC after install to only the portocols needed. \
\
&#x20;[https://support.microsoft.com/en-us/help/179442/how-to-configure-a-firewall-for-domains-and-trusts](https://support.microsoft.com/en-us/help/179442/how-to-configure-a-firewall-for-domains-and-trusts)\
&#x20;[https://isc.sans.edu/diary/Cyber+Security+Awareness+Month+-+Day+27+-+Active+Directory+Ports/7468](https://isc.sans.edu/diary/Cyber+Security+Awareness+Month+-+Day+27+-+Active+Directory+Ports/7468)

#### Step 2b: Assign Static IP

Make sure the server has already booted up and pulled an IP from DHCP. In this instance the DHCP server will be pfSense instead of the Domain Controller.\
\
Assign a static IP by opening the pfSense WebConfig. \
Status -> DHCP Leases -> identify the server and assign static mapping. To assign a static IP look for the smaller plus icon under the Actions Tab.\


#### Step 2c: Update Services under Workstations and Servers Port Group

Update the services being hosted by pfSense for the Workstations and Servers Port Group.\
Select Services -> DHCP Server -> PG\_Servers\
DHCP can be left enabled so you can manage the static IP mapping from pfSense. \
\
Go to Servers and add the DC as the DNS Server. This step is crucial as you can not join the DC if DNS is not pointed to the DC. \
\
Go to Other Options and add the gateway to be the first usable IP in the subnet.\
Also add your Domain Name under this section. This step is also crucial if you do not plan on having the DC as the DHCP Server.  \
\
Repeat the steps above for PG\_Workstations   &#x20;

### Step 3: Rename Server

Log into the DC \
Inside of the Server Manger Window -> Configure this local server -> select Computer Name -> select change button. Rename the server to a human readable name. Reboot the device after renaming it. Upon completion of the reboot run ipconfig in the terminal to verify the Static IP and Domain name appears in the information.&#x20;

### Step 4: Promote to Domain Controller

#### Step 4a: Add Roles

Server Manager -> Dashboard -> Manage -> Add roles and features wizard\
Installation type: Role-Based or feature-based installation\
Server Selection: select your DC.\
Server Roles: Select the following roles: Active Directory Domain Services and DNS Server. (you can always repeat this step if you want additional services hosted by the DC)\
Select Next, Next, Next, Install, and promote to DC. \
\
Note: At this point you will need to go back and configure DNS and create A records for any services you want to access with a FQDN i.e. firewall.domain.com

### Step 5: Joining Windows Workstation to domain&#x20;

#### Step 5a: Create User in AD (optional)

You can use the administrator credentials to domain join a workstation but this optional step is to intruduce OUs.\
&#x20;\
Create a user in AD by opening Server Manger and then selecting Tools -> Active Directory Users and Computers

![](<../.gitbook/assets/image (94).png>)

Select your Domain and at this point you can begin creating Organizational Units (OU) or simply create one user as a proof of concept and come back later to manage your OUs. \
\
In this case we will create one user under the Users Folder and you can either Right click and then select New -> User or select the new user icon on the tool bar.  Fill out the necessary info to create a user.&#x20;

![Fill out the info for a new user. ](<../.gitbook/assets/image (13).png>)

#### Step 5b: Joining the Domain

Boot up a Windows 10 Enterprise or Pro Workstation. \
Open Control Panel -> System and Security -> System\
Go to the Computer name, domain, and workgroup settings section and select Change Settings. \
Select the Change Button and enter the Domain information. Workstation Name can be changed at this point, the Computer Name will be added as a workstation in AD after joining. \
Select Join and Enter the credentials for either the admin or the user you just created. You should recieve a Welcome message, if successful.&#x20;

![Congrats, you are done for now...](<../.gitbook/assets/image (43).png>)

At a later point you should begin restricting user privilages and creating security groups. For now the goal was simply to domain join a workstation with a new user. \
\
Don't forger to return to your pfSense and begin adjusting the firewall rules to only what is needed. \
&#x20;
