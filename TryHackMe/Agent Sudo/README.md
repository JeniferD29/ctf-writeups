# Agent Sudo | TryHackMe Write-up

## Introduction

**Agent Sudo** is an easy-difficulty TryHackMe room that focuses on web enumeration, file transfer, password cracking, steganography, and Linux privilege escalation. The room guides learners through a realistic attack chain, emphasizing the importance of careful enumeration and how seemingly small clues can lead to full system compromise.

This write-up documents my approach to solving the room, explaining each step, the reasoning behind it, and the tools used along the way. The goal is not only to capture the flags but also to understand the techniques involved.


## Task 1 - Deploy the Machine

<img width="940" height="527" alt="image" src="https://github.com/user-attachments/assets/e15c0b5e-5e79-478c-bd34-5ea99a2756ea" />


1 How many open ports?

Ans: 3


Task 2 - Enumeration

After deploying the machine, the first step was to perform enumeration to identify the services running on the target. I started by scanning the machine with Nmap to discover open ports, detect service versions, and gather additional information.

nmap -sCV <TARGET_IP>


<img width="940" height="483" alt="image" src="https://github.com/user-attachments/assets/fe625344-3325-4f7d-ab27-f220e5470088" />


The scan revealed several open ports, including an HTTP service running on port 80. Since web applications are often the initial entry point, the next step was to investigate the website.

Upon opening the webpage, a message stated that agents must use their own codename as the User-Agent to access the site. This indicated that the server was likely checking the User-Agent header and returning different responses depending on its value.

<img width="906" height="296" alt="image" src="https://github.com/user-attachments/assets/444cbede-0345-4711-96f1-78c17f9f3e59" />

 How you redirect yourself to a secret page?

Ans: user-agent

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

