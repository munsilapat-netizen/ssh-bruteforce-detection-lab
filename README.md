# ssh-bruteforce-detection-lab
lab project to understand more on practical about blue team and red team
A hands-on security home lab that simulates a real-world SSH brute-force attack and builds an automated detection and prevention system against it. The project covers both the offensive (Red Team) and defensive (Blue Team) perspectives in a single isolated environment.


# Overview

SSH brute-force attacks are one of the most common threats against internet-facing servers. In this lab I built a controlled environment, launched a brute-force attack against my own machine, detected the attack through log analysis, and then deployed a layered defense that automatically blocks the attacker — proving the defense works by re-running the attack.
Everything runs on a single host using virtualization, so no external systems were ever touched.


# Objective
Set up an automated system that detects and blocks SSH brute-force attempts, and verify that it works end-to-end.


# Architecture
<img width="1920" height="1080" alt="SSH Brute-Force Detection Lab" src="https://github.com/user-attachments/assets/69511d6a-1cfc-434f-b011-d9cad0a5a721" />



Attacker: host Ubuntu machine (192.168.56.1)
Target: Ubuntu Server VM (192.168.56.101) running OpenSSH
Network: NAT (internet access) + Host-only adapter (isolated lab network)


# Tools Used
Tool	Role
VirtualBox	Virtualization / lab environment
Ubuntu Server	Target machine (SSH server)
OpenSSH	The service under attack
Hydra	Brute-force attack simulation (Red)
auth.log / journalctl	Log analysis & attack detection (Blue)
UFW	Host firewall (layer 1 defense)
Fail2ban	Automated intrusion prevention (layer 2 defense)


#procedure
1. Lab setup

 Created an Ubuntu Server VM in VirtualBox with two network adapters (NAT + Host-only). Installed OpenSSH during setup so the VM exposes port 22.

2. SSH access

 Verified connectivity (ping) between host and VM, then confirmed remote access with ssh mun@192.168.56.101.

3. Attack simulation (Red Team)
<img width="2166" height="596" alt="hydra-attack" src="https://github.com/user-attachments/assets/64b9a39c-ad19-4507-a731-ed882b604817" />

 Ran a brute-force attack against the VM's SSH service using Hydra:
 bash
 hydra -l mun -P passlist.txt ssh://192.168.56.101

 Hydra successfully guessed the password, demonstrating how a weak/known credential can be compromised.

4. Detection (Blue Team)
<img width="752" height="259" alt="image(1)" src="https://github.com/user-attachments/assets/5dc0bc23-3e00-49d2-acba-deabb0ca46f7" />
<img width="758" height="64" alt="image(2)" src="https://github.com/user-attachments/assets/b7bff6c3-b027-4308-b4ba-4b82ccc60be6" />

 Inspected the authentication log on the target to find the attack footprint:
 bash
 sudo grep "Failed password" /var/log/auth.log
 sudo grep "Failed password" /var/log/auth.log | wc -l

 Found 18 failed login attempts, all originating from 192.168.56.1 in rapid succession — the classic signature of a brute-force attack.

5. Prevention (layered defense)
<img width="936" height="381" alt="image(3)" src="https://github.com/user-attachments/assets/cd8c021b-671b-48ad-ae84-c5e3890ae4be" />

 Layer 1 — Firewall (UFW):
 bash
 sudo ufw allow ssh
 sudo ufw enable

 Layer 2 — Fail2ban: Created /etc/fail2ban/jail.local:

 ini
 [sshd]
 enabled  = true
 maxretry = 3
 findtime = 5m
 bantime  = 10m
 bash
 sudo systemctl restart fail2ban

 Rule: any IP that fails 3 times within 5 minutes is banned for 10 minutes.

6. Verification ✅
<img width="698" height="567" alt="image(4)" src="https://github.com/user-attachments/assets/49e690cc-b9b5-43f7-8c75-d36a41807a14" />
<img width="462" height="196" alt="image(5)" src="https://github.com/user-attachments/assets/2a3117b8-f580-467f-813c-a9a015b8b45c" />

 Re-ran the attack, then checked Fail2ban:
 bash
 sudo fail2ban-client status sshd
 Status for the jail: sshd
 |- Filter
 |  |- Total failed:  5
 |- Actions
    |- Currently banned: 1
    |- Banned IP list:   192.168.56.1

 The attacker's IP was automatically banned — the defense works.

# Results
Detected 18 failed SSH login attempts from a single source IP.
Deployed a two-layer defense (UFW + Fail2ban).
Fail2ban automatically banned the attacker (192.168.56.1) after exceeding the threshold.
Confirmed the ban is temporary (10 min) by design, then auto-released — avoiding permanent lockout of legitimate admins.

 What I Learned
How SSH brute-force attacks work in practice and what they look like in logs.
Reading and filtering authentication logs like a SOC analyst (grep, wc, awk).
The difference between a firewall (controls access) and Fail2ban (reacts to behavior) — and why layered defense (defense in depth) matters.
Why config overrides belong in jail.local, not jail.conf (survives package updates).
Services must be restarted to load new configuration.


# Disclaimer

All testing was performed exclusively against my own virtual machine in an isolated lab network. Never run these tools against systems you do not own or have explicit written permission to test.
