---
description: Quick overview of the Windows Data you need
---

# Getting the Windows Data You Need

### References:

[https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)\
[https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)\
We recommend you review [Windows Logging Cheat Sheet ](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5c586681f4e1fced3ce1308b/1549297281905/Windows+Logging+Cheat+Sheet\_ver\_Feb\_2019.pdf)and tune logs as needed.

## Sysmon Install through GPO

Step 1. Create a Sysmon Folder under your SYSVOL folder in your DC

![](<../.gitbook/assets/image (145).png>)

Step 2. Download Sysmon from [Microsoft ](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon#overview-of-sysmon-capabilities)and place both sysmon.exe and sysmon64.exe in&#x20;newly created Sysmon folder

Step 3. Download a sample sysmon config from [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config), rename the file to&#x20;sysmonConfig.xml and place it within the Sysmon folder

Step 4. Enter the appropriate values for your DC and FQDN in the [batch ](https://gist.github.com/silentbreaksec/8972f8c9dce151aebbef0a58313f3971)file.

Step 5. Create a GPO that will launch this batch file on startup.\
\
Navigate to Group Policy Management

![](<../.gitbook/assets/image (134).png>)

Create GPO

![](<../.gitbook/assets/image (154).png>)

Name the GPO&#x20;

![](<../.gitbook/assets/image (149).png>)

Edit GPO

![](<../.gitbook/assets/image (157).png>)

Navigate to StartUp Script

![](<../.gitbook/assets/image (142).png>)

Add Batch Script&#x20;

![](<../.gitbook/assets/image (141).png>)

Enforce newly created GPO

![](<../.gitbook/assets/image (148).png>)

Step 6. Apply the GPO to your specified OUs. \
Step 7. Force GPO Update

![](<../.gitbook/assets/image (135).png>)

If you get the following error:

![](<../.gitbook/assets/image (143).png>)

Make sure you allow the following firewall rules in your Starter GPOs\


![](<../.gitbook/assets/image (146).png>)

Another method is running the following commands:

![](<../.gitbook/assets/image (153).png>)

You'll be able to view the group policy update results in the html file\


### Second method to install Sysmon through GPO

1. Create and link the GPO as mentioned above, but this time you'll be creating a scheduled task

![](<../.gitbook/assets/image (147).png>)

2\. Name the task and let it run as System during install&#x20;

![](<../.gitbook/assets/image (140).png>)

3\. Create Trigger

![](<../.gitbook/assets/image (156).png>)

4\. Create Alert

![](<../.gitbook/assets/image (138).png>)

You can browse and add the batch file but if it crashes when the file is selected, manually type in the path to the file \\\\\<DC>\Sysvol\\\<FQDN>\\\<Sysmon Folder>\\\<SysmonInstall.bat>\
\
Step 6. Apply the GPO to your specified OUs. \
Step 7. Force GPO Update

After a restart, you should see sysmon.exe or sysmon64.exe running as a service. Logs can be found under Applications and Services -> Microsoft -> Windows -> Sysmon

## Setting up Audit Policy

Go to the Default Domain Policy and update the Audit Policies. We recommend you review [Windows Logging Cheat Sheet ](https://static1.squarespace.com/static/552092d5e4b0661088167e5c/t/5c586681f4e1fced3ce1308b/1549297281905/Windows+Logging+Cheat+Sheet\_ver\_Feb\_2019.pdf)and tune the audit policy based off of what you need/want.

![](<../.gitbook/assets/image (137).png>)

Dont worry about the log size as we will be forwarding these logs to a SIEM. Monitoring files and registry keys will also be done by sysmon.&#x20;

## Splunk Forwarder Deployment via GPO

1. Download[ Splunk Windows MSI](https://www.splunk.com/en\_us/download/splunk-enterprise.html) and Windows SDK ([Orca](https://docs.microsoft.com/en-us/windows/win32/msi/orca-exe))
2. Open the MSI file with Orca, select New transform, and add the Splunk Install Info.\


![](<../.gitbook/assets/image (133).png>)

Change AGREETOLICENSE = Yes\
Add Row -> SPLUNKUSERNAME, SPLUNKPASSWORD, DEPLOYMENT\_SERVER\
\
Additional Rows can be added based off of the flag name in [https://docs.splunk.com/Documentation/Forwarder/8.0.5/Forwarder/InstallaWindowsuniversalforwarderfromthecommandline](https://docs.splunk.com/Documentation/Forwarder/8.0.5/Forwarder/InstallaWindowsuniversalforwarderfromthecommandline)

3\. Generate Transform -> \<SplunkForwarder.mst> \
move MST and MSI to shared folder for the GPO to access.&#x20;

![](<../.gitbook/assets/image (144).png>)

4\. Create GPO!\
We are creating a Software Install GPO but need to select the Advanced Radio Button under Properties

![](<../.gitbook/assets/image (152).png>)

Add new policy, select the MSI file in the shared folder. Then modify the install with the MST file.&#x20;

![](<../.gitbook/assets/image (150).png>)

\
Verify the file path is through the network share&#x20;

![](<../.gitbook/assets/image (151).png>)

