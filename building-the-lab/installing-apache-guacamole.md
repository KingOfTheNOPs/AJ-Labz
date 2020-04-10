# Installing Apache Guacamole

.

![Show me what you got!](../.gitbook/assets/image%20%2869%29.png)

Created this guide to assist others who may need a way to access to resources remotely when your current environment may either block or restrict what resources you have.

I personally recommend using [Chrome Remote Desktop](https://remotedesktop.google.com/) as a faster method to gaining access to your home PC but if for some reason this page is blocked by your network here's this guide. I hope it helps :-\)

## End State

Access to desktop with additional resources outside of current environment

### Requirements

VMWare Workstation \(I'm Using VMWare Workstation Pro 15.x\)  Ubuntu 18.04.3 Server  Personal Domain \(I Used Google Domains to purchase mine\)  Access to Router's Admin Page  Windows 10 Pro

## STEP 1 - Set up & Install

Download Ubuntu Here: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

This guide will not cover how to create a VM, but the biggest change during VM set up was changing the network connection to bridged. This gives the VM its own IP on your network which will assist later when the firewall must be adjusted.

No changes were made during Ubuntu's set up. Once the initial install is complete download MysticRyuujin's [Install script](https://github.com/MysticRyuujin/guac-install) for a faster set up.

Download file directly from here:

```text
wget https://git.io/fxZq5
```

Make it executable:

```text
chmod +x guac-install.sh
```

Run it as root:

```text
Interactive (asks for passwords):
./guac-install.sh
```

The script will help install all of the dependencies and adds a few helpful extensions in /etc/guacamole/extensions/ \(such as two factor authentication\)

Once complete you should have access to the guac server at [http://localhost.com:8080/guacamole/](http://localhost.com:8080/guacamole/)

```text
default credentials:
User: guacadmin
Pass: guacadmin
```

Log in and create a new user. Log in with the new user and delete the default credentials.

Next step will be to install NGINX & LetsEncrypt SSL Certificate. I recommend saving the VM's state now so you can revert back in the future if needed.

I recommend if you have some time to read [http://guacamole.apache.org/](http://guacamole.apache.org/) for a better understanding of how Guacamole works and how to trouble shoot any issues you may run into.

## STEP 2 - NGINX/LetsEncrypt Setup

At this point you should purchase your domain and create an A record for your subdomain; linking the domain to your public IP.  If you're using Google Domains then head on over to the DNS tab and then go to Custom Resource Records. 

![snapshot of DNS record from Google Domain](../.gitbook/assets/image%20%2816%29.png)

### 2.A Installing NGINX

Prior to this point I would enable port forwarding from my public IP to port 80 on my Guac Server IP. You will have to enable a rule for HTTPS later so keep this in mind.

If you have the router that was provided by AT&T I enabled port forwarding under Settings &gt; Firewall &gt; NAT/Gaming. I then selected the service Apache \(80/443\) and the device selected was my Guac Server.

If you're using pfSense then go to Firewall &gt; NAT &gt; Port Forward. I restricted the source IP to be the same IP as my phone IP during testing to limit who has access to public services. I would recommend doing the same thing if you only plan on accessing your guac server from one location \(i.e work\).

Install nginx

```text
apt-get install nginx -y
```

Enable nginx on start up

```text
systemctl enable nginx
```

\(Optional\) create file link if you aren't using the default file

```text
ln -s /etc/nginx/sites-available/guacamole-proxy /etc/nginx/sites-enabled/guacamole-proxy
```

Configure /etc/nginx/sites-available/default Example of my NGINX configuration:

```text
server {

    root /var/www/html;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;
        server_name guacamoletest.domain.com;

        access_log /var/log/nginx/guac_access.log;
    error_log /var/log/nginx/guac_error.log;

    location / {
        proxy_pass http://localhost:8080/guacamole/;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_cookie_path /guacamole/ /;
    }
```

Verify config file

```text
nginx -t
```

restart service to push update

```text
systemctl restart nginx
```

You should have access to your Guac server via HTTP now  Save your VM's snapshot for a point of reference if you mess something up

### 2.B Installing TLS/SSL Cert

If you haven't already... add port forward rule for HTTPS.

Add certbot repository

```text
add-apt-repository ppa:certbot/certbot -y
```

Install Certbot

```text
apt-get install python-certbot-nginx -y
```

Generate and Install a Let's Encrypt SSL certificate

```text
certbot --nginx
Follow the prompt and select 2 when asked to redirect all traffic on port 80 to 443.
```

You should now be able to access your guacamole server from your.domain.com

## Step 3 - Access Desktop via RDP

### 3.A Enable RDP on Windows 10 Pro PC

1. Search for Advanced System Settings
2. Click Remote Tab
3. Allow Remote Access and create a User
4. Enable Remote Desktop

### 3.B Create RDP Connection on Guac

1. Log in and go to settings
2. Select connections &gt; New Connection
3. Create Name, Location, and Protocol
4. Fill in Guacamole Proxy Parameters  

   4.A. Hostname &gt; 127.0.0.1 

   4.B. Port &gt; 4822 

   4.C. Encryption &gt; None

5. Parameters 

   5.A. Hostname &gt; IP of PC \(launch terminal and enter "ipconfig" to find IP\) 

   5.B. Port &gt; 3389 

   5.C. Username &gt; username of PC \(launch terminal and enter "whoami" to find username\) 

   5.D. Password &gt; You should hopefully remember the password to your PC :-\) 

   5.E. Domain &gt; WORKGROUP by default \(launch terminal and enter "wmic computersystem get domain" to find domain\) 

   5.F. Security Mode &gt; NLA 

   5.G. Ignore Server Certificate &gt; check box

6. Save a snapshot of your VM in case the guac server breaks. \(shit happens sometimes\)

#### Example of Connection

![Guac Connection Snapshot](../.gitbook/assets/image%20%2833%29.png)

I hope this guide helps you and there are more to come.

