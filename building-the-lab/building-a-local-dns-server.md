# Building a Local DNS Server

![Looking cool because your your shit works... Its always DNS!](../.gitbook/assets/image%20%2836%29.png)

In this guide we will be covering building a BIND9 DNS server on Ubuntu 19.10 Server. For more information about Berkeley Internet Named Domain visit: [https://en.wikipedia.org/wiki/BIND](https://en.wikipedia.org/wiki/BIND)  
We utilized this DNS Server for the installation of VCSA since it requires DNS \(if you want less headaches\).   


### Step 1: Install BIND9 

```text
sudo -i 
apt-get install bind9
# verify the service is running once the install is complete
```

### Step 2: Basic Configuration

edit /etc/bind/named.conf.local  
Replace "domain" with the name of your domain

```text
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "AJ.labz"{
	type master;
	//file "/etc/bind/zones/db.domain.com";
	file "/etc/bind/zones/db.aj.labz
};

//reverse lookup zone
zone "3.2.1.in-addr.arpa" {
type master;
//file "/etc/bind/zones/rev.db.in-addr.arpa";
file "/etc/bind/zones/db.50.24.10";
};

```

Now  create the “zones” directory as specified above  
`mkdir /etc/zones  
cd /etc/zones`  
  
create the files as specified above  
`touch /etc/zones/db.aj.labz  
touch /etc/zones/db.50.24.10`  
  
edit /etc/zones/db.aj.labz

```text
$TTL 900
@ IN SOA ns1.aj.labz. admin.aj.labz. (
1 ;<serial-number>
900 ;<time-to-refresh>
900 ;<time-to-retry>
604800 ;<time-to-expire>
900 ) ; <minimum-TTL>
;List Nameservers
 IN NS ns1.aj.labz.
 IN NS ns2.aj.labz.
;address to name mapping
esxi.aj.labz. IN A 10.24.50.100
vsphere.aj.labz. IN A 10.24.50.101
ns1.aj.labz. IN A 10.24.50.2
ns2.aj.labz. IN A 10.24.50.2
```

edit /etc/zones/db.50.24.10

```text
$TTL 900
@ IN SOA ns1.aj.labz. admin.aj.labz. (
 2 ;<serial-number>
 900 ;<time-to-refresh>
 900 ;<time-to-retry>
 604800 ;<time-to-expire>
 900) ;<minimum-TTL>
; name servers
 IN NS ns1.aj.labz.
 IN NS ns2.aj.labz.
; PTR Records
101.50 IN PTR vsphere.aj.labz. ; 10.24.50.101
100.50 IN PTR esxi.aj.labz. ; 10.24.50.100
```

restart BIND9 to enforce the changes   
`/etc/init.d/bind9 restart`

### Step 3: Test DNS Server

`nslookup 10.24.50.100 10.24.50.2`

