Root Me | TryHackMe Write-up

## Introduction

Root Me is an easy-level TryHackMe room designed to introduce the fundamentals of web application penetration testing and Linux privilege escalation. The objective is to gain initial access to the target machine through a vulnerable web application, obtain a user shell, and escalate privileges to root by identifying and exploiting a misconfigured system binary.

This walkthrough documents the methodology I followed, the tools used, and the reasoning behind each step—from reconnaissance and web enumeration to exploitation and privilege escalation—to successfully capture both the user and root flags.

Task 1 – Deploy the Machine

Once the target machine is deployed, only the IP address is provided. With no prior knowledge of the environment, the first step is to perform reconnaissance to discover the exposed services, identify open ports, and gather information that can be used in the later stages of the assessment.

<img width="1702" height="556" alt="image" src="https://github.com/user-attachments/assets/3a3cb2ba-cdf1-47c5-90b4-bb88d8e43814" />

With only the target IP address available, the first step is to perform reconnaissance to identify open ports and running services.

Task 2 – Enumeration

I began by scanning the target using Nmap with the following command:

nmap -sC -sV <TARGET_IP>

Explanation:

-sC – Executes Nmap's default NSE scripts.
-sV – Detects the versions of the running services.

The scan revealed two open ports:

22/tcp – SSH
80/tcp – Apache HTTP Server (Apache 2.4.29)

The presence of an Apache web server indicates that the web application is likely the primary attack surface, making it the next area to investigate.


<img width="802" height="382" alt="image" src="https://github.com/user-attachments/assets/f828ef18-10f6-4588-9834-89140ddab2ae" />


Task 3 - Directory Enumeration

To discover hidden files and directories hosted by the web server, I performed directory enumeration using Gobuster.

gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt

Explanation:

dir – Performs directory brute-forcing.
-u – Specifies the target URL.
-w – Specifies the wordlist to use.

The common.txt wordlist, located at /usr/share/wordlists/dirb/common.txt on Kali Linux, contains commonly used directory and file names, making it suitable for initial enumeration.

The discovered directories provided valuable information about the web application and guided the next stage of the assessment.






