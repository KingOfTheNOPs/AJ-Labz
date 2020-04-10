# Virtual Networking

![](../../.gitbook/assets/image%20%2834%29.png)

## Design

{% hint style="warning" %}
Before adding virtual machines to the environment, we need to build out the network infrastructure. Design a network map for your environment. Pro Tip use Draw.io or Google Drive
{% endhint %}

Our environment was built with many enclaves throughout and used the pfsense \([Install Virtual Firewall](install-virtual-firewall.md)\) as a "firewall on a stick" for simple management. 

Once you have an idea on paper you can clearly see what you need to do. 

![](../../.gitbook/assets/image%20%281%29.png)

{% hint style="info" %}
vSphere "distributed switchs" and its networking capabilities is the main reason  for installing VCSA. For simplicity, we will add all port groups to one virtual distributed switch, and segment all traffic through VLAN tagging. When traffic from a virtual machine to another on the same ESXI host the traffic will never exit the physical interface \(or uplink\). However, if the destination is on another host or outside/needs to traverse physical hardware it will exit the physical vnic.  **with the VLAN tag associated on the port group.**

As you get more money or if you have a lot of NICs on the back of your server you should consider segmenting traffic to multiple dvswitches 
{% endhint %}

Based on the diagram above we can create the following table for the vShpere host on the left. This will be what we are configuring on the dVswitch on VCSA. You will notice vmnet missing, this is because it will remain on the standard vswitch0.

| Port Group | VLAN |
| :--- | :--- |
| pg\_WAN | un-tagged  |
| pg\_DMZ | 100 |
| pg\_LAN | un-tagged |
| pg\_RedSpace | 500 |
| pg\_Workstations | 200 |
| pg\_Servers | 300 |
| pg\_Sensors | 400 |
| pg\_ICSL1 | un-tagged |
|  |  |

{% hint style="info" %}
Once we acquire more hardware we can VLAN segment the un-tagged interfaces. Since we dont have the ability to tag all packets ingress/egress from the host we need to leave the packets un-tagged. See the [Physical Hardware](physical-hardware.md#network-equipment) for a better description
{% endhint %}

## Creating the distributed virtual switch



