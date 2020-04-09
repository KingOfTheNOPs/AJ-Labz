---
description: Installing ESXi Hyper-visor on top of dell R710
---

# ESXi

![](../../.gitbook/assets/image%20%2825%29.png)

## Installing ESXi Hyper-visor on top of dell R710

#### **1. Download ESXi and create B**ootable **media**

For this both servers we used ESXI 6.7 since it 

> Register Here: [https://my.vmware.com/web/vmware/registration](https://my.vmware.com/web/vmware/registration)
>
> Once Registered Visit: [https://www.vmware.com/products/vsphere-hypervisor.html](https://www.vmware.com/products/vsphere-hypervisor.html)

![](../../.gitbook/assets/image%20%2835%29.png)

Click "Download Now." 

Register for the product. Under the License & Download page you now have a license key for our ESXI install. 

Use Rufus bootable media writer ****to load the ESXI hypervisor ISO you just downloaded onto a USB or hard. 

![](../../.gitbook/assets/image%20%2811%29.png)

Specify an ISO image for that. Click on the browse button next to the option, and use the local file browser to pick an ISO stored on it.

You may then modify other settings to your liking. Most are fine however for most use cases, but you can change the volume label for instance, enable disk check to check for bad blocks on the USB drive, or disable quick format. 



#### 2. VMware VMvisor Boot Menu

Once you insert the ESXi USB and reboot the server.

Using iDRAC or VGA it will display a boot menu with an option to launch “ESXi Installer” as shown below.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/1-vmvisor-boot-menu.png)

#### 3. VMware ESXi Installer Loading

While the installer is loading all the necessary modules, it will display the server configuration information at the top as shown below. In this example, I was installing VMware ESXi 4.0 on a Dell PowerEdge 2950 server.

![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/2-vmware-esxi-installer-loading.png)

#### 4. New ESXi Install

Since this is a new installation of ESXi, select “Install” in the following screen.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/3-vmware-esxi-install-prompt.png)

#### 5. Accept VMware EULA

Read and accept the EULA by pressing F11.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/4-vmware-esxi-accept-eula-300x258.png)

#### 6. Select a Disk to Install VMware ESXi

VMware ESXi 4.0.0 Installer will display all available disk groups. Choose the Disk where you would like to install the ESXi. It is recommended to choose the Disk0.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/5-vmware-esxi-select-disk.png)

#### 7. Confirm ESXi Installation

Confirm that you are ready to start the install process.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/6-vmware-esxi-confirm-install-300x80.png)

#### 8. Installation in Progress

The installation process takes few minutes. While the ESXi is getting installed, it will display a progress bar as shown below.  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/7-vmware-esxi-installing.png)

#### 9. ESXi Installation Complete

You will get the following installation completed message that will prompt you to reboot the server.

![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/8-vmware-install-complete-300x201.png)

#### 10. ESXi Initial Screen

After the ESXi is installed, you’ll get the following screen where you can configure the system by pressing F2.  
  


![](https://static.thegeekstuff.com/wp-content/uploads/2010/06/10-vmware-esxi-launched1.png)

#### 10. Setting ESXi IP Address

in progress

#### 11. Determining VMNetwork Physical Adapter

in progress







