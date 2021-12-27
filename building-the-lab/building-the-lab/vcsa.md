---
description: >-
  https://docs.vmware.com/en/VMware-vSphere/6.7/vsphere-vcenter-server-67-installation-guide.pdf
---

# vCenter Server Installation



![](<../../.gitbook/assets/image (31).png>)

## vCenter install as a virtual machine

vSphere ESXI is its own bare metal hypervisor, and can operate without any appliance to orchestrate it. However, adding the vCSA allows additional capabilities: migrating VM's between server hosts, port mirroring to a physical device, and cloning virtual machines, etc.

{% hint style="info" %}
It is **highly recommend** that you have a FQDN that can be used for VCSA. For example, you may use vcsa.myserver.com and esxi.myserver.com. IOT accomplish that you can setup your own BIND9 DNS server see how we did it here [Building a Local DNS Server](../building-a-local-dns-server.md)
{% endhint %}

Start by downloading the VCSA 6.7 ISO from the Vmware website and mounting it (double click in windows).

Run the installer&#x20;

![](<../../.gitbook/assets/image (41).png>)

Click Install&#x20;

![](<../../.gitbook/assets/image (34).png>)

### **Stage 1**

![](<../../.gitbook/assets/image (29).png>)

Enter the details for your ESXI host. Notice the FQDN instead of IP address. This is not required but preferred.&#x20;

![](<../../.gitbook/assets/image (42).png>)

Click yes on the certificate warning. Give the virtual machine a name (this can be changed later)

![](<../../.gitbook/assets/image (84).png>)

Chose the option that best fits your lab (We both used tiny)

![](<../../.gitbook/assets/image (60).png>)

Select the Datastore you want the VM to be stored on.

**This is where the FQDN is important.** If you chose to only use IP address then there is a good chance you will have plenty of trouble getting through Phase 2, as well as, the web interface for VCSA (specifically VXPD) not starting due to not being able to preform a reverse lookup among other things.&#x20;

![](<../../.gitbook/assets/image (72).png>)

### Stage 2

Click finish and continue to phase 2. If you desire you can have pfsense be your NTP server and configure it in the next step.&#x20;

Here you can create a new single sign on. If you already have one you can join to that as well.

![](<../../.gitbook/assets/image (12).png>)

After that next, next, finish..&#x20;

![](<../../.gitbook/assets/image (98).png>)
