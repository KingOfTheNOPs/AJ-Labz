---
description: Just some neat tools
---

# AD Tools

### Windapsearch

`Windapsearch` is a Python script used to perform anonymous and authenticated LDAP enumeration of AD users, groups, and computers using LDAP queries. It is an alternative to tools such as `ldapsearch`, which require you to craft custom LDAP queries. We can use it to confirm LDAP NULL session authentication but providing a blank username with `-u ""` and add`--functionality` to confirm the domain functional level.

{% embed url="https://github.com/ropnop/windapsearch" %}

### ldapsearch-ad

Python3 script to quickly get various information from a domain controller through his LDAP service.

{% embed url="https://github.com/yaap7/ldapsearch-ad" %}

### PowerView

PowerView is a PowerShell tool to gain network situational awareness on Windows domains. It contains a set of pure-PowerShell replacements for various windows "net \*" commands, which utilize PowerShell AD hooks and underlying Win32 API functions to perform useful Windows domain functionality

{% embed url="https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1" %}

### PowerUp

PowerUp aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations

{% embed url="https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp" %}

### PowerUpSQL

PowerUpSQL includes functions that support SQL Server discovery, weak configuration auditing, privilege escalation on scale, and post exploitation actions such as OS command executio

{% embed url="https://github.com/NetSPI/PowerUpSQL" %}

### DAFT

This is a database auditing and assessment toolkit written in C# and inspired by [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL/wiki). Feel free to compile it yourself or download the release from [here](https://github.com/NetSPI/DAFT/releases/tag/0.9.0).

[https://github.com/NetSPI/DAFT](https://github.com/NetSPI/DAFT)

\
