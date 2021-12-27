# Virtual Networking

![](<../../.gitbook/assets/image (74).png>)

## Design

{% hint style="warning" %}
Before adding virtual machines to the environment, we need to build out the network infrastructure. Design a network map for your environment. Pro Tip use Draw.io or Google Drive
{% endhint %}

Our environment was built with many enclaves throughout and used the pfsense ([Install Virtual Firewall](install-virtual-firewall.md)) as a "firewall on a stick" for simple management.&#x20;

Once you have an idea on paper you can clearly see what you need to do.&#x20;

![](<../../.gitbook/assets/image (3).png>)

{% hint style="info" %}
vSphere "distributed switchs" and its networking capabilities is the main reason  for installing VCSA. For simplicity, we will add all port groups to one virtual distributed switch, and segment all traffic through VLAN tagging. When traffic from a virtual machine to another on the same ESXI host the traffic will never exit the physical interface (or uplink). However, if the destination is on another host or outside/needs to traverse physical hardware it will exit the physical vnic.  **with the VLAN tag associated on the port group.**

As you get more money or if you have a lot of NICs on the back of your server you should consider segmenting traffic to multiple dvswitches&#x20;
{% endhint %}

Based on the diagram above we can create the following table for the vShpere host on the left. This will be what we are configuring on the dVswitch on VCSA. You will notice vmnet missing, this is because it will remain on the standard vswitch0.

| Port Group       | VLAN       |
| ---------------- | ---------- |
| pg\_WAN          | un-tagged  |
| pg\_DMZ          | 100        |
| pg\_LAN          | un-tagged  |
| pg\_RedSpace     | 500        |
| pg\_Workstations | 200        |
| pg\_Servers      | 300        |
| pg\_Sensors      | 400        |
| pg\_ICSL1        | un-tagged  |
|                  |            |

{% hint style="info" %}
Once we acquire more hardware we can VLAN segment the un-tagged interfaces. Since we dont have the ability to tag all packets ingress/egress from the host we need to leave the packets un-tagged. See the [Physical Hardware](physical-hardware.md#network-equipment) for a better description
{% endhint %}

## Creating the distributed virtual switch

From the Networking tab, right click on your datacenter and select -> Distributed Switch -> New Distributed Switch

Chose a good name for the switch&#x20;

![](<../../.gitbook/assets/image (18).png>)

![](<../../.gitbook/assets/image (38).png>)

![](<../../.gitbook/assets/image (71).png>)

Finish

## Configure Distributed Port Group

### Create Distributed Port Group

Right click the new distributed switch you added -> Distributed Port Group -> New port group

![](<../../.gitbook/assets/image (47).png>)

Apply Configure the port group settings based on the above table

{% tabs %}
{% tab title="VLAN Tagged Port Group " %}


![](<../../.gitbook/assets/image (14).png>)
{% endtab %}

{% tab title="NON VLAN Tagged Port Group " %}
![](<../../.gitbook/assets/image (5).png>)
{% endtab %}

{% tab title="Trunk Port Group" %}
![Tune the allowed vlans default is allow 0-4094](<../../.gitbook/assets/image (10).png>)
{% endtab %}
{% endtabs %}

Click Finish

![](<../../.gitbook/assets/image (91).png>)

### Add the new D-Switch to each ESXi Host

Click on networking Networking -> your distributed switch -> actions -> add and manage hosts

![](<../../.gitbook/assets/image (36).png>)

Chose your ESXi Hosts you would like this D-Switch applied to.&#x20;

Now, assign the NIC that your switch is going to use as an uplink. Select the vnmic#, "Assign Uplink" and then pick the associated uplink on the virtual switch. Repeat as necessary, then click Next

![](<../../.gitbook/assets/image (11).png>)

{% hint style="info" %}
You can name your Uplinks to help avoid mis-configuration later &#x20;
{% endhint %}

On step 4, don't make any changes to the vmk0 VMkernel NIC.&#x20;

### Traffic Teaming and Failover

{% hint style="info" %}
ESXI hosts manage traffic shaping on a virtual switch similar to how a Cisco switch might manage a port channel or ether channel link. Traffic could go out either physical port and come back on a different port. Since we are using physical nics on the back for physical access to different enclaves this will cause issues later. The solution is explicitly define the route&#x20;
{% endhint %}

Click on the DSwitch you created and Then Topology and you can see all the NICS. Currently, ESXI will determine which port to use.&#x20;

{% hint style="info" %}
Due to VCSA only managing 1 ESXi host I created "Uplink 4" as a "null"  which has no physical NIC and I configured explicit fail order for all port groups to start to use Uplink 4
{% endhint %}

![](<../../.gitbook/assets/image (80).png>)

Now right click on the DSwitch and "Manage Distribute Port Groups"

![](<../../.gitbook/assets/image (61).png>)

Check Teaming and Failover

![](<../../.gitbook/assets/image (52).png>)

Select all Port Groups&#x20;

{% hint style="danger" %}
This is for first time configuration only. **Do not do this every time you want to change the fail order over a port group.** If we we had Physical networking equipment we would put everything in 1 Uplink and tag packets as necessary physically&#x20;
{% endhint %}

![](<../../.gitbook/assets/image (88).png>)

Change to "Use explicit failover order"

![](<../../.gitbook/assets/image (55).png>)

**Configure individual portgroups to use explicit routing** &#x20;

Right click on the port group you want to edit and click edit settings\


![](<../../.gitbook/assets/image (57).png>)

click Teaming and Failover and change the uplinks \


![](<../../.gitbook/assets/image (59).png>)

Verify using the topology and clicking on the specific uplink. \


![](<../../.gitbook/assets/image (21).png>)

## Creating a SPAN&#x20;

From the Networking tab, select the switch -> Configure -> Port Mirroring -> New

![Port Mirror](<../../.gitbook/assets/image (107).png>)

In this example, we select Distributed Port Mirroring to select specific host traffic&#x20;

![](<../../.gitbook/assets/image (104).png>)

Name the SPAN and select enable. Pick a VLAN that isn't being used.&#x20;

![SPAN Properties](<../../.gitbook/assets/image (102).png>)

Add the hosts you want to aggregate into the SPAN

![Add sources](<../../.gitbook/assets/image (108).png>)

In this instance I only added pfSense's LAN interface because I only want to monitor the traffic from my access points. Now take a look at your NSM and you'll notice you're missing some network data that you may or may not want, add additional ports to gain an understanding of how data is flowing through your network. \
\
Using this type of Port Mirror gives you the flexibility to pick and choose what you're monitoring.&#x20;

![pfSense LAN added to SPAN](<../../.gitbook/assets/image (103).png>)

For destination we selected Port ID 230. We have already installed Security Onion as our NSM in this example. Write down the MAC address as you'll need to remember which interface needs to be set to monitoring mode during SO setup. For help installing Security Onion visit [HERE](https://aj-labz.gitbook.io/aj-labz/creating-an-siem/installing-security-onion-so)

![](<../../.gitbook/assets/image (105).png>)

You're all done! Back to finding where Morty ran off to.

![](<../../.gitbook/assets/image (99).png>)
