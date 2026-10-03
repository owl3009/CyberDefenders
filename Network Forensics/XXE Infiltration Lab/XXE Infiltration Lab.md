# **XXE Infiltration Lab**

## **Tools**
Platform: CyberDefenders

Category: Network Forensics

Difficulty: Easy

Tools: Wireshark


## **Description**
Analyze PCAP data using Wireshark to identify XXE vulnerabilities, extract compromised credentials, and detect web shell uploads for persistence.

## **1. Goal**

An automated alert has detected unusual XML data being processed by the server, which suggests a potential XXE (XML External Entity) Injection attack. This raises concerns about the integrity of the company's customer data and internal systems, prompting an immediate investigation.

Analyze the provided PCAP file using the network analysis tools available to you. Your goal is to identify how the attacker gained access and what actions they took.

## **2. Key Clues**

- "An automated alert has detected unusual XML data"
- "...a potential XXE (XML External Entity) Injection attack"
- "Analyze the provided **PCAP file** using the **network analysis tools** available to you."

## **3. Plan**

- I saw a "Start here" folder"
- I looked into two folders "Artifacts" and "Tools"
- Artifacts folder contains 1 PCAP file while Tools folder shows network analysis tools
- Since I have wireshark installed already, I proceed to open XXEInfiltration.pcap from the Artifacts folder

## **4. Steps**

**Question 1: During the attacker's port scan, what is the highest-numbered TCP port that responded as open on the victim host? (Hint: in a SYN scan, the victim's SYN-ACK source port is the open port.)**

1. On the display filter, filter only SYN & ACK flags. Command: `tcp.flags.syn == 1 && tcp.flags.ack == 1`
![](Attachments/Question%201.1.png)
2. Next at the top bar, go to Statistics -> Endpoints
3. Go to TCP section and click the Packets bar showing the most scanned packets.
![](Attachments/Question%201.2.png)
4. Note that in Question 1, the hint provided shows "0000" meaning that it requires 4 digits. So it cannot be port 80 but it is port 3306 as is the second most packets.

Answer: 3306

**Question 2: By identifying the vulnerable PHP script, security teams can directly address and mitigate the vulnerability. What's the complete URI of the PHP script vulnerable to XXE Injection?**

1. Since I am looking for URI that contains a php script. The command to use is `http.request.uri contains "php"`
2. Filter based on the protocol as HTTP is not vulnerable and find any unique protocols.
3. I found HTTP/XML with php as "/review/upload.php" and the XML is the vulnerability issue.
![](Attachments/Question%202.png)

Answer: /review/upload.php

**Question 3: To construct the attack timeline and determine the initial point of compromise. What's the name of the first malicious XML file uploaded by the attacker?**

1. Based on the previous question, the vulnerable PHP script is from the HTTP/XML protocol and I will filter the packet based on that reference. The command is `http && xml`
2. Clicking on the first frame and opening the MIME and the Encapsulated multipart part: (text/xml), we find the XML file to be "TheGreatGatsby.xml" that was uploaded by the attacker.
![](Attachments/Question%203.png)

Answer: TheGreatGatsby.xml

**Question 4: Understanding which sensitive files were accessed helps evaluate the breach's potential impact. What's the name of the web app configuration file the attacker read?**

1. Using the same display filter as before, I go through each of the packets.
2. I found a frame that modifies the web's configuration.
![](Attachments/Question%204.png)

Answer: config.php

**Question 5: To assess the scope of the breach, what is the password for the compromised database user?**

1. Going through all the frames with the same filter again, I found that the same frame has an additional information from `<foo>`. There is a DB command that contains the password as `Winter2024`. 
![](Attachments/Question%205.png)

Answer: Winter2024

**Question 6: After stealing the credentials from the config file, the attacker authenticates to the MySQL service on the victim. Using the Wireshark filter `mysql.login_request`, what is the timestamp (UTC) of the attacker's first MySQL login attempt?**

1. Using the previous question frame, I started there with the display filter `frame.number >=88338` to find what the attacker did after.
2. I noticed the colour differences and the MySQL protocol with the info as "Server Greeting..."
![](Attachments/Question%206.png)

Answer: 2024-05-31 12:08

**Question 7: To eliminate the threat and prevent further unauthorized access, can you identify the name of the web shell that the attacker uploaded for remote code execution and persistence?**

1. Going back with the display filter `http && xml`, I went through all of the frames and I found a php file that also contain payload in it.
![](Attachments/Question%207.png)

Answer: booking.php
## **5. Answers**

Question 1: 3306

Question 2: /review/upload.php

Question 3: TheGreatGatsby.xml

Question 4: config.php

Question 5: Winter2024

Question 6: 2024-05-31 12:08

Question 7: booking.php

## **6. Lessons Learned**

- Read the question carefully
- Refer to the previous question as it builds up to the current question