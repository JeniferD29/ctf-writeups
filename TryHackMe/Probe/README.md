# Probe | TryHackMe Write-up

## Introduction

Probe is an easy-level TryHackMe room that focuses on reconnaissance and service enumeration. Starting with only the target's IP address, the objective is to gather information about the exposed services and answer the questions by performing systematic enumeration.

This walkthrough explains the approach I followed, the tools used, and the reasoning behind each step to successfully complete the room.


<img width="602" height="274" alt="image" src="https://github.com/user-attachments/assets/f08f1447-bf94-43f7-84b0-02545cb15926" />



## Task 1 – Deploy the Machine

After deploying the machine, the only information available is the target IP address. Since no additional details are provided, the first step is to enumerate the target and identify the services running on it.

## Task 2 – Enumeration

To begin the assessment, I performed an Nmap scan to identify open ports, detect service versions, and collect additional information about the target.

```bash
nmap -A -p- -Pn -T5 <TARGET_IP>
```

The scan revealed the exposed services and their versions, providing the information needed to answer several of the room's initial questions while guiding the next stages of enumeration.


<img width="602" height="450" alt="image" src="https://github.com/user-attachments/assets/d6c2f242-ac64-4e4a-a033-101a83db967b" />


### Question 1: What is the version of the Apache server?

The initial Nmap scan identified the web server running on the target. From the service version detection, the Apache server version was found to be:

**Answer:** `2.4.41`

### Question 2: What is the port number of the FTP service?

The same Nmap scan also revealed an FTP service running on a non-standard port. Reviewing the scan results showed that **vsftpd** was listening on port **1338**.

**Answer:** `1338`

### Question 3: What is the FQDN for the website hosted using a self-signed certificate and contains critical server information as the homepage?

The initial Nmap scan provided SSL certificate details for the HTTPS service. Inspecting the certificate's **Common Name (CN)** revealed the Fully Qualified Domain Name (FQDN) configured for the web server.

**Answer:** `dev.probe.thm`


<img width="602" height="124" alt="image" src="https://github.com/user-attachments/assets/9a6239db-6a02-4591-8653-75defe44c913" />


### Question 4: What is the email address associated with the SSL certificate used to sign the website mentioned in Q3?

Accessing the web server over HTTP returns a **403 Forbidden** response, so the HTTPS service must be used instead. 


<img width="602" height="362" alt="image" src="https://github.com/user-attachments/assets/1f0afccd-3ef6-4dea-b6df-a09817514822" />


After opening the HTTPS endpoint in a browser and viewing the website's SSL certificate, the certificate details reveal the email address associated with the self-signed certificate.


<img width="502" height="436" alt="image" src="https://github.com/user-attachments/assets/a24e0685-cc3e-42e3-a015-1702fcd924c9" />


**Answer:** `probe@probe.thm`

### Question 5: What is the value of the PHP Extension Build on the server?

During the initial enumeration, the Nmap scan revealed that the service running on **port 1443** hosted a **PHP information page (`phpinfo()`)**. 


<img width="602" height="467" alt="image" src="https://github.com/user-attachments/assets/d7480b58-ed5c-4fa0-9945-8a67d313c60c" />


Since this endpoint was accessible over HTTPS, I opened it in the browser and accepted the self-signed certificate warning.

The **phpinfo()** page exposes detailed PHP configuration information. Searching for the **PHP Extension Build** field revealed the required value.

<img width="498" height="216" alt="image" src="https://github.com/user-attachments/assets/819f103f-cbed-4ea9-9fc3-09be52cf9d70" />


**Answer:** `API20190902,NTS`


### Question 6: What is the banner for the FTP service?

The initial Nmap scan identified an FTP service running on **port 1338**. To retrieve the service banner, I connected to the port using **Netcat**, which displays the banner presented by the FTP server upon connection.

```bash
nc -nv <TARGET_IP> 1338
```

<img width="234" height="78" alt="image" src="https://github.com/user-attachments/assets/66ce847e-1d90-4a79-9d97-9ae46e882d7f" />


The server responded with the FTP welcome banner, which contained the required flag.

**Answer:** `THM{WELCOME_101113}`


### Question 7: What software is used for managing the database on the server?

To discover additional web resources, I performed directory enumeration using **Gobuster** against the web service. Among the discovered directories was **`/phpmyadmin`**, indicating that the server uses **phpMyAdmin** as its database management interface.

```bash
gobuster dir -u http://<TARGET_IP>:8000 -w /usr/share/wordlists/dirb/big.txt
```

<img width="602" height="278" alt="image" src="https://github.com/user-attachments/assets/bbe6dde8-94ba-43a7-811d-cf553a5b9048" />


The scan identified the following endpoint:

```text
/phpmyadmin
```

**Answer:** `phpmyadmin`

<img width="444" height="364" alt="image" src="https://github.com/user-attachments/assets/bd638d45-2a39-4151-b99f-8a68605252bf" />


### Question 8: What is the Content Management System (CMS) hosted on the server?

To identify the web technologies used by the application, I scanned the HTTPS service running on **port 9007** using **Nikto**.

```bash
nikto -h https://<TARGET_IP>:9007 -ssl
```

The scan revealed the `X-Redirect-By` HTTP header, which indicated that the website was running **WordPress**.

<img width="602" height="277" alt="image" src="https://github.com/user-attachments/assets/8cac0262-aa4b-4ff0-b785-2461bd8ad146" />


**Answer:** `WordPress`

### Question 9: What is the version number of the CMS hosted on the server?

The **Nikto** scan also disclosed the version of the CMS running on the web server. From the scan results, the installed **WordPress** version was identified as:

**Answer:** `6.2.2`


### Question 10: What is the username for the admin panel of the CMS?

To enumerate WordPress users, I used **WPScan**, which can identify valid usernames through various enumeration techniques.

```bash id="qh1h0s"
wpscan --url https://<TARGET_IP>:9007 --disable-tls-checks --enumerate u
```

The scan successfully identified the administrator username.

<img width="561" height="118" alt="image" src="https://github.com/user-attachments/assets/95c2cdbc-b1bd-4651-beee-a11e1a7951fa" />


**Answer:** `joomla`

### Question 11: During vulnerability scanning, OSVDB-3092 detects a file that may be used to identify the blogging site software. What is the name of the file?

The required information was obtained from the **Nikto** scan performed against the web server. Among the scan results, the **OSVDB-3092** check reported a file that could be used to identify the blogging software.

```text
+ /license.txt: License file found may identify site software.
```

<img width="602" height="23" alt="image" src="https://github.com/user-attachments/assets/aafbd1df-7326-4bc7-bbce-071129be4513" />


**Answer:** `license.txt`


### Question 12: What is the name of the software being used on the standard HTTP port?

The answer can be obtained from the initial **Nmap** scan. The results show that the HTTP service running on the standard port (**80**) is powered by **lighttpd**.


<img width="461" height="144" alt="image" src="https://github.com/user-attachments/assets/5d544f47-6dcf-418e-8383-8d7b006f1b35" />


**Answer:** `lighttpd`


### Question 13: What is the flag value associated with the web page hosted on port 8000?

Accessing the web service on **port 8000** initially displayed a blank page. To identify any hidden resources, I performed directory enumeration using **Gobuster**.

```bash
gobuster dir -u http://<TARGET_IP>:8000 -w /usr/share/wordlists/dirb/big.txt
```

The scan discovered several directories, including **`/contactus`**, **`/javascript`**, and **`/phpmyadmin`**. Upon visiting the **`/contactus`** page, the required flag was displayed.

<img width="602" height="157" alt="image" src="https://github.com/user-attachments/assets/b62dfa12-dce4-468f-b526-09aa47ae2a52" />


**Answer:** `THM{CONTACT_US_1100}`

## Conclusion

The Probe room demonstrates the importance of thorough reconnaissance and service enumeration. By systematically analyzing exposed services, web applications, and server configurations, it is possible to gather valuable information and answer each challenge without requiring exploitation of complex vulnerabilities.

## Key Learnings

- Performed comprehensive service enumeration using **Nmap**.
- Analyzed SSL certificates to gather server information.
- Enumerated web directories using **Gobuster**.
- Identified web technologies and vulnerabilities using **Nikto**.
- Enumerated WordPress users with **WPScan**.
- Retrieved service banners using **Netcat**.
- Collected information from exposed web resources and configuration pages.

**HAPPY HACKING!**
