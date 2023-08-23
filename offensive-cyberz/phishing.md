---
description: Credential Gathering - Phishing with Go Phish
---

# Phishing

## GoPhish

**Phishing Server Setup Guide:**\
For our phishing email we will be using Go Phish to create the phishing email with a link to our malicious domain. The method show was to gain sensitive information. This same method can be used to gain credentials to other accounts, but in an attempt to avoid violating the policy agreements with a company like amazon or google, we are using a local restaurant to spoof our website.

### **Step 1:** Download GoPhish&#x20;

[https://github.com/gophish/gophish/releases\
](https://github.com/gophish/gophish/releases)The prereq to this is you have the ability to port forward traffic to your own Linux VM at home or are using a VM on a VPS provider.

### **Step 2: install go phish**

Unzip the file and modify the file gophish to be an executable\
**`mkdir /opt/gophish`**\
**`unzip gophish.zip -d /opt/gophish`**\
**`chmod +x gophish`**\
**`./gophish`**

![](https://lh6.googleusercontent.com/BVrveXdkwHvWEbcaKzE\_Bex6IYZaCy0zKzTeLYVsIZ4FmoGf\_XY8qdnT9IKfQKj5oHQS\_a7GDSZOWtB29hF5mZSvR9rELGkV8bDVq0DvqMRYAcbjhtMjd-Cq3IHINnXrCrttDrb9)

### **Step 3: Purchase Domain**

Add an A record to your public IP&#x20;

![](<../.gitbook/assets/image (167).png>)

### **Step 4: Generate TLS Certificate for the site**

Install certbot and generate cert

`apt-get install certbot`\
`certbot certonly -d <domain> --manual --preferred-challenges dns`

Add TXT record with challenge information to your domain.\
Verify that the TXT record was updated and then press enter.

![Certbot cert](<../.gitbook/assets/image (160).png>)

![verify the TXT record](<../.gitbook/assets/image (161).png>)

Replace the default certificates with the newly generated ones\
First, move the new certificates to the gophish directory (optional)

![](https://lh3.googleusercontent.com/oTSbRu7VUCiLdAdAbFdQVBzBlanOjzJ74iFurWx68nPdAlI4Gw1kEACAdM5lpYJdZ6YAYeXHaXXWmJfW4Dx9a8EdWzUZrZuuwe-mhTlnp2MBqWSiqM-Po3H9c\_qjM7ilxH5wyHMl)

Second update the config.json with the new certificate names and the IP of the VM.&#x20;

![config.json](<../.gitbook/assets/image (165).png>)

Stop ./gophish and start it back up to enable the new certificates\
This will make the website look a little bit more legit. \


### Step 5: Configure phishing attack.&#x20;

**Add the targeted users:**\
Go to Users & Groups; add the targeted user and email address.&#x20;

![](https://lh3.googleusercontent.com/bYulbqiVEdnFaxiv4IesPlyQCn5KeZ3Dr-O7qWjZ8bOq9Rq9Rrw1YssQg4eH4jDJSzu0kXNI4vJnnne7FMEPhvYsWL9I5b95ZAqwelMezJZIIB81w4RqPLZUlz0sKRcwm6\_o9BZG)

**Create Email Template:**\
Go to Email Templates.\
It’s important to add the embedded URL into your email to make it look slightly more legit. Add photos and signatures as needed to make the phishing email more believable! &#x20;

![](https://lh4.googleusercontent.com/cjCTwc7QuH\_\_\_qnVg7LIF7d2kS0Yh7gxmIQCh071h1lwADqhXWBimaw-VvE7rXvO74mnAGUbKbm3uS9XiYUJKq1-euuyhkaI263vuFzzEB5G8L5hr4b-RVpsGZFCulRXDh8mn-xN)

Import the site and then add your form to capture their input

![](https://lh3.googleusercontent.com/hLneBzlCnSKGqFEg251nloK1VB-nvGdKDy00\_a0f\_ex33lFit02oVzTEdMSszuVl1PDhvM6V-6ko-IHUHyeX7YOIW361datoh31sC5qm\_EgoQq3FoOYlr7bEys55-SRUZnel0ArS)

Add email that the phishing emails will send from:

![](<../.gitbook/assets/image (162).png>)

**Enabling Gmail Account for SMTP Relay:**\
Go to myaccount.google.com and select the Security tab. From there allow less secure application access

![](<../.gitbook/assets/image (158).png>)

Test email credentials by using the Test Email button at the bottom:

![](<../.gitbook/assets/image (159).png>)

Check your email for the test email from the phishing account. Once that is verified you’re redy for the final step

### **Step 6: Create Campaign**

Head on over to the Campaign tab and fill out the campaign information.

![](<../.gitbook/assets/image (163).png>)

Key thing is to add the URL of the domain you’re trying to impersonate. aka the domain you purchased\


