# SCADA Splunk

![](.gitbook/assets/nomoney.jpg)

## Using Splunk as a SCADA

We could continue forwarding process data to our splunk using  KEPServerNX, but where is the fun in that. We want a sustained solution and just bought Splunk \#outofmoney. So we decided to use Python and stream data directly into splunk. 

Splunk has the ability to trigger external events and for purposes that will be a python script. It is can be an API call to something designed to write to a tag like an OPC server... KEPServerNX... Again have you seen my security budget? Actually, didn't buy get splunk the IT department bought it.

## SCADA with Python

{% hint style="warning" %}
This method is inherently less secure because the splunk server is using ENIP/CIP to poll the Micro850 PLC directly. A "more right" solution would be to have an intermediary server like a historian polling your PLCs and then shipping that data off to splunk through a uni-directional gateway over UDP \(See below for poc **SCADA Using OPC Server** \).
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
In this method we will be simulating a data-diode since we dont have one. All traffic to the splunk server will be over UDP. Something that can traverse something like a waterfall with ease.  
{% endhint %}

### Overview

KEPServerEX is a very versatile OPC server and has great documentation. The reason it was chosen for this lab is because we are a bit lazy. We started with dev'ing an OPC server in python with FreeOpcUa / python-opcua  locally decided to shift to KEP because the 2 hour trial wasn't and issue for us and we had a windows VM not being used on the server networked and ready to go from another prohect. Did I say we were being a bit lazy...

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

![](.gitbook/assets/image%20%28124%29.png)

#### 3. Add your TAGs

If you remember from the ladder diagram each input and output had a TAG assigned. We want to know the data 

![](.gitbook/assets/image%20%28126%29.png)

#### 4. Add splunk connection

Click "Add Splunk Connection" on the left menu bar.

Type the IP and Port you configured or will configure on splunk to receive the data.

![](.gitbook/assets/image%20%28123%29.png)

See [installing-splunk](creating-an-siem/installing-splunk.md) for help on adding connection in splunk to receive the data you are now forwarding.



