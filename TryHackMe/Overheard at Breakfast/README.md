## Overheard at Breakfast — TryHackMe CTF Walkthrough

## Introduction

Before diving into the challenge, the room presents us with a seemingly ordinary breakfast conversation. At first glance, it appears to be nothing more than a casual exchange between two individuals. However, hidden within the conversation are several clues that can be used to identify a person through publicly available information. Since no technical exploitation is involved, the challenge requires careful observation, logical analysis, and the effective use of OSINT techniques to uncover the intended attack path and ultimately retrieve the flag.



<img width="602" height="319" alt="image" src="https://github.com/user-attachments/assets/925dd1ce-5312-46ff-a045-b84c2416b27d" />

## Analyzing the Provided File

The room provides a single downloadable ZIP file. After extracting the archive, I found an image named **conversation.png**. Opening the image revealed a conversation between **Ponzi** and **Lambo**.

At first, the chat appears to be a normal conversation, but on closer inspection, it contains several pieces of information that can be used as OSINT clues. These clues will help us identify the target and continue the investigation.

<img width="354" height="105" alt="image" src="https://github.com/user-attachments/assets/641d6d25-8109-4ea4-990f-1c2c8a91f22a" />


<br>


<img width="602" height="400" alt="image" src="https://github.com/user-attachments/assets/cbfb77a0-9c93-4538-8396-21adb34ef3b7" />

## Examining the Conversation

I carefully read through the conversation and found that, although most of it was casual, it contained a few valuable OSINT clues. Two details immediately stood out:

* An email address: **[lambobytelotushotel@gmail.com](mailto:lambobytelotushotel@gmail.com)**
  
* A mention of a free platform that allows users to upload a profile and link their social media accounts, with its name starting with the letter **"G"**.

These clues provided a good starting point for the investigation.

My first step was to investigate the email address to see if it revealed any publicly available information. However, searching the email directly did not return any useful results, so I decided to focus on the second clue instead.

Since searching the email address did not reveal any useful information, I shifted my focus to the second clue mentioned in the conversation. It referred to a free platform that allows users to create a profile and link their social media accounts, with its name starting with the letter **"G"**.

Based on this hint, I searched for profile platforms that matched the description.


The search results pointed me to **Gravatar**, a service that lets users create a public profile associated with their email address. This matched the clue perfectly, so I proceeded with Gravatar to continue the investigation.


<img width="602" height="179" alt="image" src="https://github.com/user-attachments/assets/02559b12-bc1b-49e6-8aa4-50d62ca90a6f" />



## Finding the Gravatar Profile

Since the previous clue pointed towards **Gravatar**, I visited the website and searched using the email address found in the conversation.

Gravatar links user profiles to their email addresses by generating a unique hash. Instead of manually creating the hash, I used Gravatar's email lookup feature, entered the email address from the conversation, and searched for the associated profile.

The search successfully returned **Lambo's Gravatar profile**, along with the profile URL and other publicly available information. With the correct profile identified, I could continue investigating the linked details to find the information required to complete the challenge.


<img width="602" height="503" alt="image" src="https://github.com/user-attachments/assets/d96a02be-53ea-4bd3-b346-e98727186617" />


After opening the Gravatar profile, I examined the publicly available information and noticed that the profile biography contained a long encoded-looking string

The text resembled a Base64-encoded value, suggesting that it might contain the hidden flag or another important clue. The next step was to decode this string and see what information it revealed.


<img width="530" height="352" alt="image" src="https://github.com/user-attachments/assets/f75eea44-92fb-495f-8b84-23c24e03d884" />


To determine what the encoded text contained, I copied the string from the Gravatar profile biography and pasted it into CyberChef

I then applied the From Base64 operation to decode the value. The decoded output revealed the room flag, successfully completing the challenge.



<img width="602" height="263" alt="image" src="https://github.com/user-attachments/assets/aa8db9c7-526f-4154-9b25-ae2ef0959041" />




## Conclusion

This room demonstrated how valuable **Open Source Intelligence (OSINT)** can be when investigating publicly available information. By carefully analyzing a simple conversation, following the available clues, identifying a Gravatar profile, and decoding the hidden Base64 string, I was able to successfully retrieve the flag. The challenge highlights the importance of attention to detail and how small pieces of information can be connected to uncover hidden data.

## Key Learnings

* Analyzing conversations for useful OSINT clues.
* Identifying valuable information such as email addresses and profile hints.
* Using **Gravatar** to investigate publicly available profiles.
* Recognizing and decoding **Base64-encoded** data using **CyberChef**.
* Understanding how publicly shared information can reveal more than intended.

**HAPPY HACKING!**




