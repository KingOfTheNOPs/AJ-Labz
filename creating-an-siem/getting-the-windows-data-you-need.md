---
description: Quick overview of the Windows Data you need
---

# Getting the Windows Data You Need

![](../.gitbook/assets/win_data_2.jpg)

### References:

[https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)  
[https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon\#overview-of-sysmon-capabilities](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)  
We recommend you review [Windows Logging Cheat Sheet ](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5c586681f4e1fced3ce1308b/1549297281905/Windows+Logging+Cheat+Sheet_ver_Feb_2019.pdf)and tune the logs you need as well.

## Sysmon Install through GPO

Step 1. Create a Sysmon Folder under your SYSVOL folder in your DC

![](../.gitbook/assets/image%20%28145%29.png)

Step 2. Download Sysmon from [Microsoft ](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)and place both sysmon.exe and sysmon64.exe in newly created Sysmon folder

Step 3. Download a sample sysmon config from [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config), rename the file to sysmonConfig.xml and place it within the Sysmon folder

Step 4. Enter the appropriate values for your DC and FQDN in the [batch ](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)file.

Step 5. Create a GPO that will launch this batch file on startup.  
  
Navigate to Group Policy Management

![](../.gitbook/assets/image%20%28134%29.png)

Create GPO

![](../.gitbook/assets/image%20%28154%29.png)

Name the GPO 

![](../.gitbook/assets/image%20%28149%29.png)

Edit GPO

![](../.gitbook/assets/image%20%28157%29.png)

Navigate to StartUp Script

![](../.gitbook/assets/image%20%28142%29.png)

Add Batch Script 

![](../.gitbook/assets/image%20%28141%29.png)

Enforce newly created GPO

![](../.gitbook/assets/image%20%28148%29.png)

Step 6. Apply the GPO to your specified OUs.   
Step 7. Force GPO Update

![](../.gitbook/assets/image%20%28135%29.png)

If you get the following error:

![](../.gitbook/assets/image%20%28143%29.png)

Make sure you allow the following firewall rules in your Starter GPOs  


![](../.gitbook/assets/image%20%28146%29.png)

Another method is running the following commands:

![](../.gitbook/assets/image%20%28153%29.png)

You'll be able to view the group policy update results in the html file  


### Second method to install Sysmon through GPO

1. Create and link the GPO as mentioned above, but this time you'll be creating a scheduled task

![](../.gitbook/assets/image%20%28147%29.png)

2. Name the task and let it run as System during install 

![](../.gitbook/assets/image%20%28140%29.png)

3. Create Trigger

![](../.gitbook/assets/image%20%28156%29.png)

4. Create Alert

![](../.gitbook/assets/image%20%28138%29.png)

You can browse and add the batch file but if it crashes when the file is selected, manually type in the path to the file \\&lt;DC&gt;\Sysvol\&lt;FQDN&gt;\&lt;Sysmon Folder&gt;\&lt;SysmonInstall.bat&gt;  
  
Step 6. Apply the GPO to your specified OUs.   
Step 7. Force GPO Update

After a restart, you should see sysmon.exe or sysmon64.exe running as a service. Logs can be found under Applications and Services -&gt; Microsoft -&gt; Windows -&gt; Sysmon

## Setting up Audit Policy

Go to the Default Domain Policy and update the Audit Policies. We recommend you review [Windows Logging Cheat Sheet ](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5c586681f4e1fced3ce1308b/1549297281905/Windows+Logging+Cheat+Sheet_ver_Feb_2019.pdf)and tune the audit policy based off of what you need/want.

![](../.gitbook/assets/image%20%28137%29.png)

Dont worry about the log size as we will be forwarding these logs to a SIEM. Monitoring files and registry keys will also be done by sysmon. 

## Splunk Forwarder Deployment via GPO

1. Download[ Splunk Windows MSI](https://www.splunk.com/en_us/download/splunk-enterprise.html) and Windows SDK \([Orca](https://docs.microsoft.com/en-us/windows/win32/msi/orca-exe)\)
2. Open the MSI file with Orca, select New transform, and add the Splunk Install Info. 

![](../.gitbook/assets/image%20%28133%29.png)

Change AGREETOLICENSE = Yes  
Add Row -&gt; SPLUNKUSERNAME, SPLUNKPASSWORD, DEPLOYMENT\_SERVER  
  
Additional Rows can be added based off of the flag name in [https://docs.splunk.com/Documentation/Forwarder/8.0.5/Forwarder/InstallaWindowsuniversalforwarderfromthecommandline](https://docs.splunk.com/Documentation/Forwarder/8.0.5/Forwarder/InstallaWindowsuniversalforwarderfromthecommandline)

3. Generate Transform -&gt; &lt;SplunkForwarder.mst&gt;   
move MST and MSI to shared folder for the GPO to access. 

![](../.gitbook/assets/image%20%28144%29.png)

4. Create GPO!  
We are creating a Software Install GPO but need to select the Advanced Radio Button under Properties

![](../.gitbook/assets/image%20%28152%29.png)

Add new policy, select the MSI file in the shared folder. Then modify the install with the MST file. 

![](../.gitbook/assets/image%20%28150%29.png)

  
Verify the file path is through the network share 

![](../.gitbook/assets/image%20%28151%29.png)



