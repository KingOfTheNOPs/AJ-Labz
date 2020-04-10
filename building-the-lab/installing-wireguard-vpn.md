---
description: >-
  WireGuard® is an extremely simple yet fast and modern VPN that utilizes
  state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more
  useful than IPsec, while avoiding the massive hea
---

# Installing WireGuard VPN

![](../.gitbook/assets/image%20%2845%29.png)

Want to watch YouTube TV but your local channels aren't available? Try WireGuard VPN to bypass their location services. They noticed my VPN when using OpenVPN :-\(

Or use WireGuard as an alternative way to connect to your home lab with a VPN

### Install Dependancies

Install Ubuntu Server  
Install [WireGuard](https://www.wireguard.com/install/) on Ubuntu Server   
Install [WireGuard](https://www.wireguard.com/install/) on Client

If Ubuntu &gt;19.10

`sudo apt install wireguard`

If Ubuntu &lt; 19.10

`sudo add-apt-repository ppa:wireguard/wireguard    
sudo apt-get update    
sudo apt-get install wireguard`

### Configure WireGuard Server

Generate a public and private key  
`mkdir -p /etc/wireguard/keys  
cd /etc/wireguard/keys  
umask 044  
wg genkey | tee privatekey | wg pubkey > publickey`  
  
Create the WireGuard config file and add the contents below  
`/etc/wireguard/wg0.conf`

```text
[Interface]
PrivateKey = <Private Key>
Address = 10.100.100.1/24, 
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE; 
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens160 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; 
SaveConfig = true
```

The Address field is the range of IPs you will be assigning to the clients. When the client connects, the iptables rule handles the NAT in order to provide the client the IP address of the server. Saveconfig allows the new peer to be added to the config file when the service is running.   


### Set Up Firewall Rules

Make sure to port forward 51820 UDP on pfSense under Firewall -&gt; NAT 

![](../.gitbook/assets/image%20%2863%29.png)

Allow the server/clients access to the internet  
Firewall -&gt; Rules

![](../.gitbook/assets/image%20%285%29.png)

### Start WireGuard Service

Start WireGuard  
`wg-quick up wg0`  
  
Enable start up on boot  
`systemctl enable wg-quick@wg0`  
  
Verify if VPN connection is listening \(should see listening connection\)  
`wg show`

### Configure WireGuard Client

WireGuard should have been installed on the client by now, if not go to [https://www.wireguard.com/install/](https://www.wireguard.com/install/)  
On Windows: Open the application, Click the dropdown next to add tunnel and select empty tunnel.   
Add the following contents  


![](../.gitbook/assets/image%20%2851%29.png)

You should be able to connect to your home network now.   
To verify if the connection was successful run `wg show` on the wireguard server and you should see the peer information.   


![](../.gitbook/assets/image%20%2846%29.png)

