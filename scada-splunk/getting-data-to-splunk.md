---
description: >-
  This page is dedicated to configuring and installing means for getting data to
  a spunk server.
---

# Getting Data To Splunk

![](../.gitbook/assets/image%20%28125%29.png)

## SCADA with Python

{% hint style="warning" %}
This method is inherently less secure because the splunk server is using ENIP/CIP to poll the Micro850 PLC directly. A "more right" solution would be to have an intermediary server like a historian polling your PLCs and then shipping that data off to splunk through a uni-directional gateway over UDP \(See below for poc **SCADA Using OPC Server** below\).
{% endhint %}

Once you're strong enough, save the world:

{% code title="hello.sh" %}
```bash
# Ain't no code for that yet, sorry
echo 'You got to trust me on this, I saved the world'
```
{% endcode %}

## SCADA Using OPC Server 

{% hint style="info" %}
In this method we will be simulating a data-diode since we don't actually have one. All traffic to the splunk server will be over UDP. This protocol is connection-less meaning it can traverse something like a waterfall data-diode with ease.  
{% endhint %}

### Overview

KEPServerEX is a very versatile OPC server \(plus lots more\)  and has great documentation. The reason it was chosen for this lab is because we are a bit lazy. We started with dev'ing an OPC server in python with FreeOpcUa / python-opcua  locally decided to shift to KEP because the 2 hour trial wasn't and issue for us and we had a windows VM not being used on the server networked and ready to go from another project. Did I say we were being a bit lazy...

### Installing KEPServerEX

Visit [https://www.kepware.com/en-us/products/kepserverex/](https://www.kepware.com/en-us/products/kepserverex/) and click on Download Free Demo

Next...Next-Finish

### Configuring KEPServerEX

#### 1. Create a new channel

Right-click on Connectivity and create a new channel. We called ours Micro800

For the Channel we chose "Allen-Bradley Micro800 Ethernet" because the PLC we are using is a Micro800 yours might differ.

#### 2. Create a new device

Now it is time to add the PLC you want KEP to poll for the TAGs we will define later. 

Richt-click the new channel and add device. Name it and input the IP of the PLC.

Leave the rest default 

![](../.gitbook/assets/image%20%28126%29.png)

#### 3. Add your TAGs

If you remember from the ladder diagram each input and output had a TAG assigned. We want to know the data 

![](../.gitbook/assets/image%20%28128%29.png)

#### 4. Add splunk connection

Click "Add Splunk Connection" on the left menu bar.

Type the IP and Port you configured or will configure on splunk to receive the data.

![](../.gitbook/assets/image%20%28123%29.png)

See [installing-splunk](../creating-an-siem/installing-splunk.md) for help on adding connection in splunk to receive the data you are now forwarding.

## Sending SEL Data to Splunk 

coming soon. 

