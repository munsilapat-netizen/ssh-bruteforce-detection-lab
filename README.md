# ssh-bruteforce-detection-lab
lab project to understand more on practical about blue team and red team
A hands-on security home lab that simulates a real-world SSH brute-force attack and builds an automated detection and prevention system against it. The project covers both the offensive (Red Team) and defensive (Blue Team) perspectives in a single isolated environment.


Overview

SSH brute-force attacks are one of the most common threats against internet-facing servers. In this lab I built a controlled environment, launched a brute-force attack against my own machine, detected the attack through log analysis, and then deployed a layered defense that automatically blocks the attacker — proving the defense works by re-running the attack.
Everything runs on a single host using virtualization, so no external systems were ever touched.


Objective
Set up an automated system that detects and blocks SSH brute-force attempts, and verify that it works end-to-end.


Architecture




Attacker: host Ubuntu machine (192.168.56.1)
Target: Ubuntu Server VM (192.168.56.101) running OpenSSH
Network: NAT (internet access) + Host-only adapter (isolated lab network)
