# Agent Sudo | TryHackMe Write-up

## Introduction

**Agent Sudo** is an easy-difficulty TryHackMe room that focuses on web enumeration, file transfer, password cracking, steganography, and Linux privilege escalation. The room guides learners through a realistic attack chain, emphasizing the importance of careful enumeration and how seemingly small clues can lead to full system compromise.

This write-up documents my approach to solving the room, explaining each step, the reasoning behind it, and the tools used along the way. The goal is not only to capture the flags but also to understand the techniques involved.


## Task 1 - Deploy the Machine

<img width="940" height="527" alt="image" src="https://github.com/user-attachments/assets/e15c0b5e-5e79-478c-bd34-5ea99a2756ea" />



## Task 2 - Enumeration

After deploying the machine, the first step was to perform enumeration to identify the services running on the target. I started by scanning the machine with Nmap to discover open ports, detect service versions, and gather additional information.

nmap -sCV <TARGET_IP>


<img width="940" height="483" alt="image" src="https://github.com/user-attachments/assets/fe625344-3325-4f7d-ab27-f220e5470088" />


The scan revealed several open ports, including an HTTP service running on port 80. Since web applications are often the initial entry point, the next step was to investigate the website.

Upon opening the webpage, a message stated that agents must use their own codename as the User-Agent to access the site. This indicated that the server was likely checking the User-Agent header and returning different responses depending on its value.

<img width="906" height="296" alt="image" src="https://github.com/user-attachments/assets/444cbede-0345-4711-96f1-78c17f9f3e59" />



After opening the website in the browser, the homepage displayed a message indicating that **agents must use their own codename as the User-Agent** to access the site. This suggested that the web server was checking the HTTP **User-Agent** header and responding differently based on its value.

The message mentioned the codename **R**, so I decided to test it by sending a request with a modified User-Agent using `curl`.

```bash
curl -A "R" -L http://<TARGET_IP>
```

The response confirmed that **`R`** is a valid employee codename, but it was **not the one required** to access the hidden content.

Since the hint referred to **25 employees** and there are **26 letters in the English alphabet**, it was reasonable to assume that each employee was assigned a **single-letter codename**. Based on this observation, I decided to test each letter of the alphabet as the **User-Agent** value, starting from **`A`**, until a different response was returned.

<img width="784" height="419" alt="image" src="https://github.com/user-attachments/assets/5967c2d5-f5d3-4b0f-ab90-db31f21e34a7" />


Testing the **`B`** User-Agent produced the same response as before, indicating that it was not the correct codename.

However, when I changed the **User-Agent** to **`C`**, the server returned a different response. This confirmed that **`C`** was the correct agent codename and revealed new information that could be used in the next stage of the attack.


<img width="940" height="130" alt="image" src="https://github.com/user-attachments/assets/14971ed2-d254-4ba0-8719-eef696ee6339" />


## Task 3 - Hash Cracking and Brute Force

The first objective in this task is to obtain the **FTP password**. Since a valid username was discovered during the enumeration phase and the previous hint mentioned that the password was **weak**, the next step was to perform a brute-force attack against the FTP service.

For this, I used **Hydra**, a popular password-cracking tool that can test multiple passwords against network services. The **`rockyou.txt`** wordlist was chosen because it contains millions of commonly used passwords and is widely used in CTFs and penetration testing labs.

```bash
hydra -l <USERNAME> -P /usr/share/wordlists/rockyou.txt ftp://<TARGET_IP>
```

### Explanation

- `-l` specifies the username obtained during the enumeration phase.
- `-P` provides the path to the password wordlist (`rockyou.txt`).
- `ftp://<TARGET_IP>` tells Hydra to target the FTP service running on the machine.

After a short time, Hydra successfully identified the correct FTP password, allowing access to the FTP server and enabling further enumeration.

<img width="940" height="151" alt="image" src="https://github.com/user-attachments/assets/fdab438a-0536-45eb-976e-2232fb6266ea" />

With the FTP credentials successfully recovered, the next step was to log in to the **FTP server** and enumerate its contents. The goal was to identify any files that could contain sensitive information or provide clues for further exploitation.

After connecting to the server, I listed the available files and downloaded them to my local machine for analysis.

```bash
ftp <TARGET_IP>
```

<img width="940" height="250" alt="image" src="https://github.com/user-attachments/assets/b45e4aad-484c-4367-ade7-3b20f08874f5" />


The next questions mention a ZIP file and a steganography password, neither of which we have at this stage. This suggests that one of the downloaded files may contain hidden data. Since image files are commonly used for steganography, I used Binwalk on the PNG file to check for any embedded files.

<img width="926" height="478" alt="image" src="https://github.com/user-attachments/assets/5a9f968d-efc8-4ab7-9377-62348f82b799" />


<img width="591" height="238" alt="image" src="https://github.com/user-attachments/assets/d2d886dc-e19c-4651-a9f6-57ff1d55e04b" />


The extracted ZIP file was password-protected, so the next step was to recover its password. I first used **`zip2john`** to extract the ZIP hash and then used **John the Ripper** to crack the hash and obtain the password.

<img width="940" height="348" alt="image" src="https://github.com/user-attachments/assets/38eb87e8-3fcf-490c-8cc0-a5b7d71b2bfd" />


Now if we see the extracted text file we get this

<img width="507" height="97" alt="image" src="https://github.com/user-attachments/assets/3007de2f-fd66-4adf-9817-7d22edd567c4" />


The extracted text appeared to contain the required information, but it was encoded. To decode it, I used CyberChef, which can automatically detect common encoding formats. In this case, CyberChef identified the text as Base64 and successfully decoded it.


<img width="940" height="433" alt="image" src="https://github.com/user-attachments/assets/7c1cb849-2112-456f-9890-9d81fcc49021" />


After decoding the text, I obtained the passphrase Area51. The only remaining file was a JPG image, and since Steghide is commonly used to hide data inside JPEG images using a passphrase, it was likely that this was the required steganography password. To confirm this, I checked the image with Steghide, which revealed that it contained embedded data.

<img width="590" height="251" alt="image" src="https://github.com/user-attachments/assets/57988827-875c-42d1-b342-bffee662c4cc" />


After extracting it with the password we found.

<img width="910" height="306" alt="image" src="https://github.com/user-attachments/assets/588493e8-2030-4695-b3fe-0fc146440d87" />


## Task 4 - Capture the User Flag

Using the credentials obtained from the previous task, I logged in to the target machine via SSH. After accessing the user's home directory, I found two files, one of which contained the user flag.



<img width="876" height="715" alt="image" src="https://github.com/user-attachments/assets/7c5bb04e-7738-4cd2-a9b3-ab3bf1a84bb7" />


The second file was an image named Alien_autospy.jpg. To answer the related question, I transferred the image from the target machine to my local system using scp and then performed a reverse image search to identify its origin.

scp <user@TARGET_IP>:Alien_autospy.jpg /local/directory/



## Task 5 - Privilege Escalation

The final task is to perform privilege escalation and obtain root access. To begin, I used the standard enumeration commands to check the permissions and privileges assigned to the current user.

<img width="940" height="176" alt="image" src="https://github.com/user-attachments/assets/496ec780-c2e8-421a-8312-4c73c136d44d" />


The output showed that the current user was not allowed to run /bin/bash as root, as indicated by the !root restriction. However, the ALL entry suggested that the user could execute /bin/bash as any other user, which appeared unusual and hinted at a possible privilege escalation opportunity.

A quick search for this sudo configuration revealed a known technique that could be used to exploit this behavior and gain root privileges.

<img width="940" height="350" alt="image" src="https://github.com/user-attachments/assets/a14e94d7-199c-42b0-8f91-cf674bc7d74b" />


A closer look at the installed Sudo version revealed that it was 1.8.27, which is vulnerable to CVE-2019-14287. This vulnerability allows a user with Runas ALL permissions to bypass the !root restriction by specifying a crafted user ID, effectively executing commands with root privileges. This made it possible to escalate privileges and gain full access to the system.

Since the installed Sudo version was older than 1.8.28, the system was vulnerable to CVE-2019-14287. Using the identified exploit, I was able to bypass the restriction, spawn a root shell, and successfully retrieve the root flag.

<img width="940" height="334" alt="image" src="https://github.com/user-attachments/assets/737330cf-2cb4-496d-9631-300ba12443d8" />



## Conclusion

The **Agent Sudo** room provided a practical introduction to several fundamental penetration testing techniques. Starting with web enumeration, the challenge progressed through User-Agent manipulation, FTP brute forcing with Hydra, file analysis using Binwalk and Steghide, password cracking with John the Ripper, and concluded with privilege escalation by exploiting the **CVE-2019-14287** Sudo vulnerability.

### Key Learnings

- Performed service enumeration using **Nmap**.
- Manipulated HTTP **User-Agent** headers to uncover hidden content.
- Brute-forced FTP credentials using **Hydra** and the **rockyou.txt** wordlist.
- Identified and extracted hidden files using **Binwalk** and **Steghide**.
- Recovered passwords using **John the Ripper**.
- Decoded Base64-encoded data with **CyberChef**.
- Transferred files securely using **SCP**.
- Escalated privileges by exploiting **CVE-2019-14287** to obtain root access.

**HAPPY HACKING!**
