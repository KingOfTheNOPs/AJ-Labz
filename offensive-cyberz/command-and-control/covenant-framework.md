# Covenant Framework

![](<../../.gitbook/assets/image (51).png>)

## Install Using Dotnet Core

The easiest way to use Covenant is by installing dotnet core. You can download dotnet core for your platform from here.

Be sure to install the dotnet core version 3.1 SDK!

### Install Dotnet core version 3.1

```
wget -q https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1
```

Be sure to clone Covenant recursively to initialize the git submodules: `git clone --recurse-submodules https://github.com/cobbr/Covenant`&#x20;

### Download and Install Covenant

Once you have installed dotnet core, we can build and run Covenant using the dotnet CLI:

```
cd Covenant/Covenant
dotnet build
dotnet run
```

Now, open a browser and surf to https://0.0.0.0:7443

Accept the security warning

At this point you will create an admin user for Covenant.

Enter a username and password

And that’s it, Covenant is ready for use!
