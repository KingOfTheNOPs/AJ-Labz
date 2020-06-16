# Installing Protocol Analyzers

![](../.gitbook/assets/parse.png)

### Security Onion Prerequisites  

#### Update security onion 

```bash
sudo soup
```

#### Install GCC-6

```bash
sudo apt-get update && \
sudo apt-get install build-essential software-properties-common -y && \
sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y && \
sudo apt-get update && \
sudo apt-get install gcc-6 g++-6 -y && \
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 60 --slave /usr/bin/g++ g++ /usr/bin/g++-6 && \
gcc -v
```

#### Install cmake by a PPA \(Upgrade to 3.2\)

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:george-edison55/cmake-3.x
sudo apt-get update
sudo apt-get install cmake
```

#### Install Package libpcap0.8-dev package

```bash
sudo apt-get install libpcap0.8-dev
```

This should cover any errors you may encounter 

### Install and Configure ZKG Command-Line Utilitiy

Install `zkg`  
Using the latest stable release on [PyPI](https://pypi.python.org/pypi):

```text
$ pip install zkg
```

Or using the latest git development version:

```text
$ pip install git+git://github.com/zeek/package-manager@master
```

{% hint style="info" %}
If not using something like **virtualenv** to manage Python environments, the default user script directory is `~/.local/bin` and you may have to modify your `PATH` to search there for **zkg**.
{% endhint %}

After installing via **pip**, additional configuration is required. First, make sure that the **zeek-config** script that gets installed with **zeek** is in your `PATH`. Then, as the user you want to run **zkg** with, do:

```text
$ zkg autoconfig
```

This automatically generates a config file with the following suggested settings that should work for most Zeek deployments:

* script\_dir: set to the location of Zeek's `site` scripts directory \(e.g. _`<zeek_install_prefix>`_`/share/zeek/site`\)
* plugin\_dir: set to the location of Zeek's default plugin directory \(e.g. _`<zeek_install_prefix>`_`/lib/zeek/plugins`\)
* zeek\_dist: set to the location of Zeek's source code. If you didn't build/install Zeek from source code, this field will not be set, but it's only needed if you plan on installing packages that have uncompiled Zeek plugins.

{% hint style="info" %}
 If your Zeek installation is owned by "root" and you intend to run **zkg** as a different user, then you should grant "write" access to the directories specified by script\_dir and plugin\_dir. E.g. you could do something like:

```text
$ sudo chgrp $USER $(zeek-config --site_dir) $(zeek-config --plugin_dir)
$ sudo chmod g+rwX $(zeek-config --site_dir) $(zeek-config --plugin_dir)
```
{% endhint %}

 The final step is to edit your `site/local.zeek`. If you want to have Zeek automatically load the scripts from all [installed](https://docs.zeek.org/projects/package-manager/en/stable/zkg.html#install-command) packages that are also marked as "[loaded](https://docs.zeek.org/projects/package-manager/en/stable/zkg.html#load-command)" add:

```text
@load packages
```

### Install Third-Party Plugin for ZEEK using the ZKG Utility

Start by cloning the git repo of the package you want to install

```bash
git clone https://github.com/amzn/zeek-plugin-enip
cd zeek-plugin-enip
CWD="$(pwd)"
```

Use the `zkg` tool to install the plugin.

```bash
zkg install --force --skiptests $CWD
```

This script will loop through the listed repos and install  install the plugins with `zkg` 

```bash
#!/bin/bash

# going to clone under /usr/local/src
SRC_BASE_DIR="/usr/local/src"
mkdir -p "$SRC_BASE_DIR"

#
# get_latest_github_tagged_release
#
# get the latest GitHub release tag name given a github repo URL
#
function get_latest_github_tagged_release() {
  REPO_URL="$1"
  REPO_NAME="$(echo "$REPO_URL" | sed 's|.*github\.com/||')"
  LATEST_URL="https://github.com/$REPO_NAME/releases/latest"
  REDIRECT_URL="$(curl -fsSLI -o /dev/null -w %{url_effective} "$LATEST_URL" 2>/dev/null)"
  if [[ "$LATEST_URL" = "$REDIRECT_URL"/latest ]]; then
    echo ""
  else
    echo "$REDIRECT_URL" | sed 's|.*tag/||'
  fi
}

#
# clone_github_repo
#
# clone the latest GitHub release tag if available (else, master/HEAD) under $SRC_BASE_DIR
#
function clone_github_repo() {
  REPO_URL="$1"
  if [[ -n $REPO_URL ]]; then
    REPO_LATEST_RELEASE="$(get_latest_github_tagged_release "$REPO_URL")"
    SRC_DIR="$SRC_BASE_DIR"/"$(echo "$REPO_URL" | sed 's|.*/||')"
    rm -rf "$SRC_DIR"
    if [[ -n $REPO_LATEST_RELEASE ]]; then
      git -c core.askpass=true clone --branch "$REPO_LATEST_RELEASE" --depth 1 "$REPO_URL" "$SRC_DIR" >/dev/null 2>&1
    else
      git -c core.askpass=true clone --depth 1 "$REPO_URL" "$SRC_DIR" >/dev/null 2>&1
    fi
    [ $? -eq 0 ] && echo "$SRC_DIR" || echo "cloning \"$REPO_URL\" failed" >&2
  fi
}

# install Zeek packages that insatll nicely using zkg
ZKG_GITHUB_URLS=(
  https://github.com/amzn/zeek-plugin-bacnet
  https://github.com/amzn/zeek-plugin-enip
  https://github.com/amzn/zeek-plugin-profinet
  https://github.com/amzn/zeek-plugin-s7comm
  https://github.com/amzn/zeek-plugin-tds
  #https://github.com/salesforce/ja3
)
for i in ${ZKG_GITHUB_URLS[@]}; do
  SRC_DIR="$(clone_github_repo "$i")"
  [[ -d "$SRC_DIR" ]] && zkg install --force --skiptests "$SRC_DIR"
done
```

### Manual Install

Start by cloning the git repo of the package you want to install

Then follow the steps below

```bash
git clone https://github.com/amzn/zeek-plugin-enip
cd zeek-plugin-enip
./configure 
make && \ make install

# if plugin_dir && zeek_dist are not in path you will need to manually specify them
#./configure --bro-dist="zeek_dist" --install-root="plugin_dir"
```

![](../.gitbook/assets/image%20%2873%29.png)



