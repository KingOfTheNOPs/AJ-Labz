# Industrial Control Systems \(ICS\)

![](../.gitbook/assets/ezgif-1-d3acee8c463e.gif)

## Physically

The main PLC is the Micro850 \(the one with the sticker\), however, we plan to interoperate the  Micrologix1100 as a producer of a tag or two in a process we have yet to think of... 

![](../.gitbook/assets/plc_setup.jpg)

No wiring diagram created but the Black and Green buttons are NO and the Red button is NC. 

## Process Logic

The process logic on this PLC has changed many many times, but at the time of this photo a simple four rung ladder program was created to latch the light upon push of the respective color button and unlatch by pushing the black. 

![](../.gitbook/assets/program.png)

As you can see we make use of Rockwell latching/unlatching \(or set/reset in CCW\) blocks. Since the red button is NC we need to XIO and the other two buttons are NO to we XIC.

## KepWare  

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

![](../.gitbook/assets/image%20%28124%29.png)

#### 3. Add your TAGs

If you remember from the ladder diagram each input and output had a TAG assigned. We want to know the data 

![](../.gitbook/assets/image%20%28126%29.png)

#### 4. Add splunk connection

Click "Add Splunk Connection" on the left menu bar.

Type the IP and Port you configured or will configure on splunk to receive the data.

![](../.gitbook/assets/image%20%28123%29.png)

See [installing-splunk](../creating-an-siem/installing-splunk.md) for help on adding connection in splunk to receive the data you are now forwarding.



