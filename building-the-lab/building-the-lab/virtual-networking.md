# Virtual Networking

![](../../.gitbook/assets/image%20%2863%29.png)

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

From the Networking tab, right click on your datacenter and select -&gt; Distributed Switch -&gt; New Distributed Switch

Chose a good name for the switch 

![](../../.gitbook/assets/image%20%2815%29.png)

![](../../.gitbook/assets/image%20%2832%29.png)

![](../../.gitbook/assets/image%20%2861%29.png)

Finish

## Configure Distributed Port Group

### Create Distributed Port Group

Right click the new distributed switch you added -&gt; Distributed Port Group -&gt; New port group

![](../../.gitbook/assets/image%20%2841%29.png)

Apply Configure the port group settings based on the above table

{% tabs %}
{% tab title="VLAN Tagged Port Group " %}


![](../../.gitbook/assets/image%20%2811%29.png)
{% endtab %}

{% tab title="NON VLAN Tagged Port Group " %}
![](../../.gitbook/assets/image%20%283%29.png)
{% endtab %}

{% tab title="Trunk Port Group" %}
![Tune the allowed vlans default is allow 0-4094](../../.gitbook/assets/image%20%287%29.png)
{% endtab %}
{% endtabs %}

Click Finish

![](../../.gitbook/assets/image%20%2877%29.png)

### Add the new D-Switch to each ESXi Host

Click on networking Networking -&gt; your distributed switch -&gt; actions -&gt; add and manage hosts

![](../../.gitbook/assets/image%20%2830%29.png)

Chose your ESXi Hosts you would like this D-Switch applied to. 

Now, assign the NIC that your switch is going to use as an uplink. Select the vnmic\#, "Assign Uplink" and then pick the associated uplink on the virtual switch. Repeat as necessary, then click Next

![](../../.gitbook/assets/image%20%288%29.png)

{% hint style="info" %}
You can name your Uplinks to help avoid mis-configuration later  
{% endhint %}

On step 4, don't make any changes to the vmk0 VMkernel NIC. 

### Traffic Teaming and Failover

{% hint style="info" %}
ESXI hosts manage traffic shaping on a virtual switch similar to how a Cisco switch might manage a port channel or ether channel link. Traffic could go out either physical port and come back on a different port. Since we are using physical nics on the back for physical access to different enclaves this will cause issues later. The solution is explicitly define the route 
{% endhint %}

Click on the DSwitch you created and Then Topology and you can see all the NICS. Currently, ESXI will determine which port to use. 

{% hint style="info" %}
Due to VCSA only managing 1 ESXi host I created "Uplink 4" as a "null"  which has no physical NIC and I configured explicit fail order for all port groups to start to use Uplink 4
{% endhint %}

![](../../.gitbook/assets/image%20%2867%29.png)

Now right click on the DSwitch and "Manage Distribute Port Groups"

![](../../.gitbook/assets/image%20%2852%29.png)

Check Teaming and Failover

![](../../.gitbook/assets/image%20%2844%29.png)

Select all Port Groups 

{% hint style="danger" %}
This is for first time configuration only. **Do not do this every time you want to change the fail order over a port group.** If we we had Physical networking equipment we would put everything in 1 Uplink and tag packets as necessary physically 
{% endhint %}

![](../../.gitbook/assets/image%20%2874%29.png)

Change to "Use explicit failover order"

![](../../.gitbook/assets/image%20%2846%29.png)

**Configure individual portgroups to use explicit routing**  

Right click on the port group you want to edit and click edit settings  


![](../../.gitbook/assets/image%20%2848%29.png)

click Teaming and Failover and change the uplinks   


![](../../.gitbook/assets/image%20%2850%29.png)

Verify using the topology and clicking on the specific uplink.   


![](../../.gitbook/assets/image%20%2817%29.png)

## Creating a SPAN 

