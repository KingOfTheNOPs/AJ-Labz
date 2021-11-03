# Install Virtual Machine

![](<../../.gitbook/assets/image (22) (1).png>)

###

### Step 1: Download your ISO

Feel free to download any ISO of your choosing. In this example we are using Security Onion as the VM of choice. \
\
Repository of ISOs:\
[https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)\
[https://www.centos.org/download/](https://www.centos.org/download/)\
[https://www.kali.org/downloads/](https://www.kali.org/downloads/)\
[https://www.microsoft.com/en-us/software-download/windows10](https://www.microsoft.com/en-us/software-download/windows10)\
[https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016?filetype=ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016?filetype=ISO)

### Step 2: Import into Datastore

Now that the network infrastructure is in place and you've downloaded the ISOs for your home lab, we can begin install virtual machines. Datastores are vSphere's location for virtual machine data, as well as ISOs. Upload your virtual machine ISO's to the datastore. Select Datastores -> Datastores -> Select the datastore associated with your ESXI host.\
\
Select New folder and create a directory for your ISOs. In our case we created a folder called ISOs. Select the folder you created and select upload files. Upload each ISO you plan on using.&#x20;

### Step 3: Create VM

Head on back to Hosts and Clusters and deploy a new VM.

![](<../../.gitbook/assets/image (115).png>)

In this instance we are creating a new VM.&#x20;

![](<../../.gitbook/assets/image (114).png>)

Name your VM and select the datacenter that will save the VM's data

![](<../../.gitbook/assets/image (117).png>)

![](<../../.gitbook/assets/image (113).png>)

Select compatabiliy.&#x20;

![](<../../.gitbook/assets/image (111).png>)

Select the OS and Version. In this instance we choose Ubuntu Linux 64 bit for Security Onion

![](<../../.gitbook/assets/image (120).png>)

Customize Hardware. Look up the recommended specs for your VM. \
After that make sure to select the port group that alligns with your network layout.\
\
To select the ISO, change the CD/DVD Drive option to datastore and then select your ISO from the folder that it was uploaded to.  \


![](<../../.gitbook/assets/image (109).png>)

If additional hardware is needed, like an additional NIC, now is your chance to select Add New Device. \


![](<../../.gitbook/assets/image (110).png>)

And you're ready to head on over to the VM and power on. The rest of the install instructions will vary on OS.&#x20;
