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

Getting there. 

