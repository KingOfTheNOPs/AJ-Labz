---
description: Quick overview of the Windows Data you need
---

# Getting the Windows Data You Need

### References:

[https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)  
[https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon\#overview-of-sysmon-capabilities](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)  
We recommend you review [Windows Logging Cheat Sheet ](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5c586681f4e1fced3ce1308b/1549297281905/Windows+Logging+Cheat+Sheet_ver_Feb_2019.pdf)and tune the logs you need as well.

## Sysmon Install through GPO

Step 1. Create a Sysmon Folder under your SYSVOL folder in your DC

![](../.gitbook/assets/image%20%28132%29.png)

Step 2. Download Sysmon from [Microsoft ](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)and place both sysmon.exe and sysmon64.exe in newly created Sysmon folder

Step 3. Download a sample sysmon config from [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config), rename the file to sysmonConfig.xml and place it within the Sysmon folder

Step 4. Enter the appropriate values for your DC and FQDN in the [batch ](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)file.

Step 5. Create a GPO that will launch this batch file on startup.

Step 6. Apply the GPO to your specified OUs. 

