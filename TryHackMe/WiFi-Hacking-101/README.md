# TryHackMe - WiFi Hacking 101 Walkthrough

## Introduction

This walkthrough documents the steps taken to complete the WiFi Hacking 101 room on TryHackMe. The room introduces the fundamentals of wireless security, including WPA/WPA2 authentication, monitor mode, packet capturing, and offline password cracking using the Aircrack-ng suite. It provides hands-on experience with the workflow used during wireless security assessments in a controlled lab environment.


## Task 1: The Basics — An Introduction to WPA

### 1. What type of attack on the encryption can you perform on WPA(2)-Personal?

WPA/WPA2-Personal uses a shared Pre-Shared Key (PSK) for authentication. An attacker can capture the WPA 4-way handshake during a client's connection and perform an offline password-cracking attack by testing numerous passwords from a wordlist or through brute-force techniques until the correct key is discovered.

**Answer:** `Brute Force`

---

### 2. Can this method be used to attack WPA2-EAP handshakes? (Yea/Nay)

No. WPA2-Enterprise (WPA2-EAP) cannot be attacked using the same offline brute-force method as WPA2-Personal. Unlike WPA2-Personal, it authenticates each user with unique credentials through an authentication server, making a captured handshake insufficient for a straightforward brute-force attack.

**Answer:** `Nay`

---

### 3. What is the three-letter abbreviation for the pre-shared key used in Wi-Fi security?

In WPA/WPA2-Personal networks, devices authenticate using a shared secret known as a **Pre-Shared Key (PSK)**. This key is configured on both the wireless access point and client devices, allowing them to establish a secure connection.

**Answer:** `PSK`

---

### 4. What’s the minimum length of a WPA2-Personal password?

According to the WPA2 security standard, a pre-shared key must contain at least **8 characters**. This minimum length helps provide a basic level of protection against password-guessing and brute-force attacks.

**Answer:** `8`

---

## Task 2: You're Being Watched — Capturing Packets to Attack

### 1. How do you put the interface `wlan0` into monitor mode with Aircrack tools?

Before capturing wireless traffic, the wireless adapter must be switched from **Managed Mode** to **Monitor Mode**. This allows the adapter to listen to all packets transmitted over the air instead of communicating with a single access point.

```bash
airmon-ng start wlan0
```

**Answer:** `airmon-ng start wlan0`

---

### 2. What is the new interface name likely to be after you enable monitor mode?

After monitor mode is enabled, Aircrack-ng typically creates a new virtual monitor interface by appending **`mon`** to the original interface name.

**Answer:** `wlan0mon`

---

### 3. What do you do if other processes are currently trying to use that network adapter?

Background services such as **NetworkManager** and **wpa_supplicant** may interfere with monitor mode. To avoid conflicts, terminate these processes before capturing packets.

```bash
airmon-ng check kill
```

**Answer:** `airmon-ng check kill`

---

### 4. What tool from the Aircrack-ng suite is used to create a capture?

The **`airodump-ng`** tool is used to discover nearby wireless networks and capture wireless traffic. It displays information such as the access point's **BSSID**, channel, encryption type, signal strength, and connected clients. Captured packets are saved for later analysis or password-cracking attempts.

**Answer:** `airodump-ng`

---

### 5. What flag do you use to set the BSSID to monitor?

The **`--bssid`** flag specifies the MAC address of the target access point. This filters the capture so that only packets from the selected network are recorded.

**Answer:** `--bssid`

---

### 6. What flag is used to specify the wireless channel?

The **`--channel`** flag tells **airodump-ng** which Wi-Fi channel to monitor. Since an access point operates on a specific channel, selecting the correct one ensures that all relevant traffic is captured.

**Answer:** `--channel`

---

### 7. How do you save captured packets to a file?

The **`-w`** flag specifies the output filename where captured packets will be stored. These files can later be analyzed or used with tools such as **Aircrack-ng** to perform offline password-cracking attacks.

**Answer:** `-w`


## Task 3: Aircrack-ng — Let's Get Cracking

### 1. What flag is used to specify the BSSID to attack?

The **`-b`** flag specifies the **BSSID (MAC address)** of the target wireless access point. This ensures that Aircrack-ng attempts to crack the handshake captured from the selected network.

**Answer:** `-b`


<img width="417" height="243" alt="image" src="https://github.com/user-attachments/assets/74dc48cb-91a3-46f8-bc6b-6f0b8e36b6f7" />


---

### 2. What flag is used to specify a wordlist?

The **`-w`** flag is used to provide the path to a password wordlist. During the cracking process, Aircrack-ng tests each password in the wordlist against the captured WPA/WPA2 handshake.

**Answer:** `-w`



<img width="420" height="226" alt="image" src="https://github.com/user-attachments/assets/50b72587-8b57-45bf-b5f8-be9cec8027e8" />


---

### 3. How do we create an HCCAPX file for use with Hashcat?

The **`-j`** option converts a captured WPA/WPA2 handshake into the **HCCAPX** format, which is compatible with Hashcat for GPU-accelerated password cracking.

**Answer:** `-j`


<img width="422" height="164" alt="image" src="https://github.com/user-attachments/assets/3f85daaa-f677-45b7-9128-54d628c94b2a" />



---

### 4. Using the RockYou wordlist, crack the password in the attached capture.

The following steps were performed to recover the WPA2 password from the provided capture file.

#### Step 1: Extract the provided capture archive

The downloaded capture file was extracted to obtain the wireless capture (`.cap`) file.



<img width="322" height="102" alt="image" src="https://github.com/user-attachments/assets/64b60ac5-e953-4279-9873-62bd921fe2c3" />


---

#### Step 2: Convert the capture file to HCCAPX format

The capture file was converted into **HCCAPX** format using Aircrack-ng.

```bash
aircrack-ng -j wifi capture.cap
```

During the conversion, useful information such as the **BSSID** and **ESSID** of the target network was displayed.


<img width="461" height="590" alt="image" src="https://github.com/user-attachments/assets/12d420d3-db17-4044-af36-ae352edf2ca5" />



---

#### Step 3: Crack the WPA2 handshake

Using the generated **HCCAPX** file and the **RockYou** wordlist, Aircrack-ng performed an offline dictionary attack to recover the wireless password.

```bash
aircrack-ng -a2 \
-b <BSSID> \
-w /usr/share/wordlists/rockyou.txt \
wifi.hccapx
```


<img width="594" height="366" alt="image" src="https://github.com/user-attachments/assets/b506cf22-b8ec-4a91-97d2-68c179108f50" />



After testing the passwords contained in the wordlist, the correct WPA2 pre-shared key was successfully recovered.

**Answer:** `greeneggsandham`


---

### 5. Where is password cracking likely to be fastest, CPU or GPU?

Password-cracking tools such as **Hashcat** perform significantly faster on a **GPU** because graphics processors contain thousands of parallel processing cores capable of testing many password candidates simultaneously. In comparison, CPUs have fewer cores and are generally less efficient for large-scale password-cracking tasks.

**Answer:** `GPU`



<img width="602" height="474" alt="image" src="https://github.com/user-attachments/assets/8892178a-4275-4b00-ad80-230e39d372e9" />


## Conclusion

This room provided hands-on experience with wireless penetration testing by covering WPA/WPA2 authentication, monitor mode, packet capture, and offline password cracking using the Aircrack-ng suite. It also reinforced the importance of strong passwords and secure wireless configurations.

## Key Learnings

* Understanding WPA/WPA2 authentication.
* Enabling monitor mode with `airmon-ng`.
* Capturing wireless traffic using `airodump-ng`.
* Using Aircrack-ng flags (`-b`, `-w`, `-j`, `--bssid`, `--channel`).
* Converting capture files to HCCAPX format.
* Recovering WPA2 passwords using a dictionary attack.
* Recognizing the performance advantage of GPU-based password cracking.


Happy Hacking!
