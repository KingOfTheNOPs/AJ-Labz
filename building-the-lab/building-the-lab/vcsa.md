---
description: >-
  https://docs.vmware.com/en/VMware-vSphere/6.7/vsphere-vcenter-server-67-installation-guide.pdf
---

# vCenter Server Installation



![](../../.gitbook/assets/image%20%2831%29.png)

## vCenter install as a virtual machine

vSphere ESXI is its own bare metal hypervisor, and can operate without any appliance to orchestrate it. However, adding the vCSA allows additional capabilities: migrating VM's between server hosts, port mirroring to a physical device, and cloning virtual machines, etc.

{% hint style="info" %}
It is **highly recommend** that you have a FQDN that can be used for VCSA. For example, you may use vcsa.myserver.com and esxi.myserver.com. IOT accomplish that you can setup your own BIND9 DNS server see how we did it here [Building a Local DNS Server](../building-a-local-dns-server.md)
{% endhint %}

Start by downloading the VCSA 6.7 ISO from the Vmware website and mounting it \(double click in windows\).

Run the installer 

![](../../.gitbook/assets/image%20%2841%29.png)

Click Install 

![](../../.gitbook/assets/image%20%2834%29.png)

### **Stage 1**

![](../../.gitbook/assets/image%20%2829%29.png)

Enter the details for your ESXI host. Notice the FQDN instead of IP address. This is not required but preferred. 

![](../../.gitbook/assets/image%20%2842%29.png)

Click yes on the certificate warning. Give the virtual machine a name \(this can be changed later\)

![](../../.gitbook/assets/image%20%2882%29.png)

Chose the option that best fits your lab \(We both used tiny\)

![](../../.gitbook/assets/image%20%2860%29.png)

Select the Datastore you want the VM to be stored on.

**This is where the FQDN is important.** If you chose to only use IP address then there is a good chance you will have plenty of trouble getting through Phase 2, as well as, the web interface for VCSA \(specifically VXPD\) not starting due to not being able to preform a reverse lookup among other things. 

![](../../.gitbook/assets/image%20%2872%29.png)

### Stage 2

Click finish and continue to phase 2. If you desire you can have pfsense be your NTP server and configure it in the next step. 

Here you can create a new single sign on. If you already have one you can join to that as well.

![](../../.gitbook/assets/image%20%2812%29.png)

After that next, next, finish.. 

![](../../.gitbook/assets/image%20%2895%29.png)

