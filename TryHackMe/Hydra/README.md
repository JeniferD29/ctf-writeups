# TryHackMe - Hydra Walkthrough

## Introduction

This walkthrough documents the steps taken to complete the Hydra room on TryHackMe. The room focuses on using Hydra to perform password brute-force attacks against web authentication forms.

## Task 1: Hydra Overview

Hydra is a popular password-cracking tool used to perform brute-force and dictionary attacks against various network services and web applications.

## Task 2: Using Hydra

### Accessing the Login Page

The target IP address was opened in a web browser, revealing a login page.

<img width="940" height="625" alt="image" src="https://github.com/user-attachments/assets/f01b593f-78e2-4a9c-8330-e5dd720bd0bc" />



### Brute-Forcing the Web Login

The login form uses the `/login` endpoint. To identify the password for the user `molly`, Hydra was used with the `http-post-form` module and the RockYou password list.

```bash
hydra -l molly -P Tools/wordlists/rockyou.txt <TARGET_IP> http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
```
### Command Explanation

- `-l molly` specifies the target username.
- `-P Tools/wordlists/rockyou.txt` specifies the password wordlist.
- `http-post-form` targets a web login form using HTTP POST requests.
- `/login` is the authentication endpoint.
- `username=^USER^&password=^PASS^` defines the form parameters.
- `F=incorrect` indicates a failed login attempt.

Once valid credentials were identified, they were used to successfully authenticate to the application and complete the challenge.

<img width="940" height="125" alt="image" src="https://github.com/user-attachments/assets/c9a6c685-0e0a-4b35-b314-0875b382bea5" />

Use the credentials to log in.


<img width="615" height="771" alt="image" src="https://github.com/user-attachments/assets/b17262fa-8b40-4a3d-8a5f-f0ae3e2ff238" />



<img width="940" height="360" alt="image" src="https://github.com/user-attachments/assets/85763882-387f-474d-80a1-b510df35544c" />


### Brute-Forcing SSH

The same username was tested against the SSH service using Hydra.

```bash
hydra -l molly -P Tools/wordlists/rockyou.txt <TARGET_IP> -t 4 ssh
```

<img width="940" height="200" alt="image" src="https://github.com/user-attachments/assets/290eaee7-b31c-45de-bdf8-06ee2f0c33de" />


### Connecting via SSH

After obtaining valid credentials, an SSH connection was established.

```bash
ssh molly@<TARGET_IP>
```


<img width="784" height="703" alt="image" src="https://github.com/user-attachments/assets/9bc4be64-50ab-4d0c-8d84-0e0bfd66b9df" />




<img width="453" height="126" alt="image" src="https://github.com/user-attachments/assets/c3cce4f6-87a5-48bb-adcb-28ec7571927b" />


## Conclusion

This room provided hands-on experience with Hydra and demonstrated how password brute-force attacks can be performed against both web applications and SSH services. It also highlighted the importance of strong passwords and secure authentication mechanisms.

### Key Learnings

- Using Hydra against HTTP POST login forms
- Identifying login failure conditions
- Brute-forcing SSH authentication
- Understanding Hydra command syntax and modules

Thank you for reading!

Happy Learning and Happy Hacking!







