# Install Virtual Firewall

![](../../.gitbook/assets/image%20%2861%29.png)

## Configuring PFSense W/ a LAN and VMnet 

The objective of this is to give controlled access to the lab from the home wifi. Security was lessened because, in theory, an adversary can pwn a box on the home network \(like an IOT device\) and LM to the VMnet. Once on VMNET they can own the bare-metal hypervisor and if you use iDRAC on the same VMNet then they can own your R710 too \(because I know you are too lazy to change that calvin password\).

{% hint style="info" %}
In order to complete this you will need to know how to add attach port groups to this VM. If you need help doing this please see the [Install Virtual Machine](install-virtual-machine.md) and [Virtual Networking](virtual-networking.md) pages for the requisite knowledge
{% endhint %}

### Interface Configuration

After attaching your `wan` `LAN` and `VMnet` port groups to your virtual machine you should be able to verify their status via the virtual console of PFsense. If DHCP is enabled, the LAN interface should draw a DHCP lease from your VMnetwork or the MGMT interface. If it does not you may need to statically configure one using option 2 of the pfsense cli.

![](../../.gitbook/assets/image%20%2864%29.png)

{% hint style="info" %}
 Most issues are due to improper porgroups tied to the wrong interface on the VM. Verify the interface by checking the MAC address in Vsphere and from the terminal \(option 8 and ifconfig \|less \).
{% endhint %}

### Verifying the MAC address and port groups

Add the interfaces and verify the mac address and port groups are what you want them to be. The interface mac address are in vSphere where you added the interfaces. 

_Your MAC addresses will be different. This is where a lot of troubleshooting will happen if you select the wrong interface._

![From PFsense interfaces tab. NOTE: WAN MAC](../../.gitbook/assets/image%20%2862%29.png)

![From the edit settings page of vCenter Notice the WAN Mac Address is the same.](../../.gitbook/assets/image%20%2842%29.png)

{% hint style="info" %}
On VSphere you may have many port groups that will tag packets with the specific VLANs. Also, do to interface restrictions on PFsense it is recommended to create a Trunk port group and use`VLAN` interfaces \(can be thought of as sub-interfaces\) in pfsense.
{% endhint %}

### Assigning IP Addresses to Interfaces

Configure the interfaces to have an IP address on its desired subnet. Set the configuration type to none **on Trunk Interface**. We need sub interfaces for each enclave south of the firewall. Interface -&gt; Assignments Click “LAN”

![](../../.gitbook/assets/image%20%2815%29.png)

![](../../.gitbook/assets/image%20%2836%29.png)

{% hint style="info" %}
If you are unable to access this page you will need to use the CLI \(see above\) to configure the interface IP and ensure you are in the same network. Pro Tip: Configure the VMNET interface and put your laptop in vmnet.
{% endhint %}

### Create sub-interfaces

You will need to determine your trunking interface. It will be the mac address of the interface connected to your trunk port group. 

 Go to interfaces -&gt; assignments -&gt; VLANs. Once there click add at the bottom of the page.

![Creating a sub-interface for our Planners Enclave which will pass traffic tagged with VLAN 203](../../.gitbook/assets/image%20%2870%29.png)

![These sub-interfaces will be tied to the port groups  ](../../.gitbook/assets/image%20%281%29.png)

Go back to the interfaces page and add the VLAN \(Sub\) interfaces as a network port and assign corresponding ip address Go to interfaces -&gt; assignments. Once there, click the drop down to add each new interface.

![](../../.gitbook/assets/image%20%2825%29.png)

Change the name, configuration type, and IP address the say way we did before. Keep in mind this will be the default gateway for each specific enclave when assigning the IP address.

###  Create Firewall rules

You will not need to configure any rules on the trunk link, all rules should be configured on the specific VLAN interface.

Creating firewall rules is time consuming because you need to have a deep understanding of the network flow and that pfSense uses Ingress rules to make decisions. That is an important note to think about… pfSense will process a rule you set as traffic enters the port. \(Helpful: [https://www.thegeekpub.com/8000/pfsense-rules-not-working/](https://www.thegeekpub.com/8000/pfsense-rules-not-working/)\)

Here are some windows default domain ports that will need to be open on the client VLAN interfaces: [https://support.microsoft.com/en-us/help/179442/how-to-configure-a-firewall-for-domains-and-trusts](https://support.microsoft.com/en-us/help/179442/how-to-configure-a-firewall-for-domains-and-trusts) [https://isc.sans.edu/diary/Cyber+Security+Awareness+Month+-+Day+27+-+Active+Directory+Ports/7468](https://isc.sans.edu/diary/Cyber+Security+Awareness+Month+-+Day+27+-+Active+Directory+Ports/7468)

![Example restricted rule set for AD enclave](../../.gitbook/assets/image%20%2876%29.png)

{% hint style="info" %}
Some standard practice rules you can use, though by no means exhaustive: 

* DNS is only authorized to WAN from your domain controller or your 
* DNS server Nothing allowed to ICS enclaves that aren't actually ICS 
* Don't allow VPN's outbound from user machines if it's against organization policy \(so the defenders can monitor traffic\) 
* Don't allow your DC to talk to internet \(except for DNS, or to Microsoft update services for DC updates\)
{% endhint %}

### Troubleshoot Firewall rules

If you are having issues in the domain, you can try a couple quick steps to try to fix them.

Here is an issue I came across. I was attempting to push a GPO across the domain and it would not work after my firewall rules were in place. I attempted to access \rebels.sw\sysvol and it did not work. This meant I had an error or was mission a NETLOGON or File Services rule, but was unsure which one.

I added a allow any any rule at the bottom of my rule set so it would be processed last. Then I attempted to connect to sysvol and it worked.

![](../../.gitbook/assets/image%20%2849%29.png)

So now I can look at the connection state of the any any rule \(click the blue 9/69kib\) and see if I can determine if there is something I need to change. NOTE: you should probably see more than 0

![](../../.gitbook/assets/image%20%283%29.png)

![](../../.gitbook/assets/image%20%2832%29.png)

This lets me know I should probably add a tcp rule allowing port 88 for Kerberos authentication services.

