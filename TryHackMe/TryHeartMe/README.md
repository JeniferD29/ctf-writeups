# TryHeartMe — TryHackMe CTF Walkthrough

## Introduction
Before diving into the challenge, the application appeared to be a Valentine's Day–themed online gift shop filled with chocolates,
teddy bears, flowers, and heart-shaped gifts. However, it quickly became apparent that there were no available credits to purchase any items, 
and there was no visible feature or functionality that allowed users to earn or add credits through normal interaction. This suggested that further enumeration would be required to uncover the intended attack path.


<img width="1445" height="727" alt="image" src="https://github.com/user-attachments/assets/a34e6bbb-f0ec-46e6-91b9-1b36d63ce075" />
<br>

Let’s start this room and see how we got the flag, first of all let’s just do an Nmap scan to see if there is any open ports which could help us in sorting this CTF.

<img width="806" height="395" alt="image" src="https://github.com/user-attachments/assets/95a24d6e-84cf-49ab-a747-f69931218096" />
<br>


After doing the Nmap scan we got only 2 open ports: 
ssh,http

With the target identified, visit port 5000 and start enumerating the web application.

<img width="1896" height="592" alt="image" src="https://github.com/user-attachments/assets/ee617cd2-6838-4067-9c42-c09ba2125235" />
<br>


As the website is designed as a Valentine's gift shop, creating a user account is the logical first step. After registering, we can explore the application's functionality and continue the enumeration process.


<img width="497" height="426" alt="image" src="https://github.com/user-attachments/assets/95ded396-1300-4eb3-b03f-e8f1d08c294a" />
<br>

After creating an account, I explored the application by viewing one of the available products to better understand its functionality. While inspecting the product details, I noticed an interesting field displaying the value `role:user`. This immediately suggested that the application might be storing or exposing user role information in a way that could potentially be manipulated. If the role value could be modified from `user` to `admin`, it might lead to privilege escalation. Based on this observation, the next step was to investigate how the application processed this parameter and determine whether it was possible to alter it.



<img width="1137" height="447" alt="image" src="https://github.com/user-attachments/assets/aa899661-201c-4333-bca3-8081e17cb90d" />
<br>


Since the `role:user` value seemed interesting, I right-clicked on the page and selected **Inspect**, then navigated to the **Cookies** section to examine how the application stored the user role. The goal was to determine whether modifying the role from `user` to `admin` could lead to privilege escalation.


<img width="1607" height="611" alt="image" src="https://github.com/user-attachments/assets/f0114648-6c14-4007-9827-c6384493b5dc" />
<br>

This revealed that the application was using a **JSON Web Token (JWT)**. JWT vulnerabilities often arise from weak secret keys, improper signature validation, or insecure token handling. Since JWTs are commonly used for authentication and session management, exploiting these weaknesses can lead to privilege escalation or even account takeover.


First of all we’ll copy the value here and add it in jwt.io, which acts a decoder to the JWT sessions.

<img width="1667" height="590" alt="image" src="https://github.com/user-attachments/assets/9ac16391-9bd3-43b0-a5c5-611f26fdae42" />
<br>

The decoded JWT contains several claims, including the user's role. Since the token currently identifies our role as `user`, it raises the possibility of modifying this value to `admin`. To inspect the token further, copy the JWT and decode it using **CyberChef**.


<img width="1907" height="486" alt="image" src="https://github.com/user-attachments/assets/55502e97-8cbe-4a8c-a8d9-9da79cbee3d3" />
<br>

As shown in the screenshot above, I modified the `role` claim from `user` to `admin` in an attempt to gain administrative privileges. After re-encoding the JWT, I copied the updated token and replaced the existing session value in the website's cookies.


<img width="1915" height="577" alt="image" src="https://github.com/user-attachments/assets/aa11dba7-bb88-441c-8fd8-73dc69bd1b6a" />
<br>

Now paste it there and refresh the page.

<img width="1181" height="521" alt="image" src="https://github.com/user-attachments/assets/1dea5602-c802-47e6-97b7-f3755b41dddc" />
<br>

And here we gooo you’re now the admin!! Let’s find our flag now.
click on admin



<img width="1192" height="307" alt="image" src="https://github.com/user-attachments/assets/4d40fe37-9fdd-491c-938d-5a7542525791" />
<br>

Click on Open ValenFlag.

<img width="1210" height="492" alt="image" src="https://github.com/user-attachments/assets/447db177-f617-43c3-92f5-39301344002e" />
<br>

Click on Buy.


<img width="1196" height="337" alt="image" src="https://github.com/user-attachments/assets/a95fbf39-4ab8-409b-b1a0-3fddb702fdac" />
<br>

## Conclusion

This room demonstrated how an insecure JWT implementation can lead to privilege escalation. By modifying the JWT role claim and replacing the session token, administrative access was obtained and the flag was captured.

## Key Learnings

* Identifying JWT authentication
* Decoding and modifying JWTs
* Understanding JWT-based privilege escalation

**Happy Hacking!**



