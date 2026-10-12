# **RedLine Lab**

## **Tools**
Platform: CyberDefenders

Category: Endpoint Forensics

Difficulty: Easy

Tools: Volatility, Strings


## **Description**
Employ Volatility to analyze a memory dump, identifying suspicious processes, network IOCs, memory protections, and attacker's command-and-control infrastructure.

## **1. Goal**

As a member of the Security Blue team, your assignment is to analyze a memory dump using Redline and Volatility tools. Your goal is to trace the steps taken by the attacker on the compromised machine and determine how they managed to bypass the Network Intrusion Detection System (NIDS). Your investigation will identify the specific malware family employed in the attack and its characteristics. Additionally, your task is to identify and mitigate any traces or footprints left by the attacker.

## **2. Key Clues**

- "...analyze a memory dump **using Redline and Volatility tools**."
- Your goal is to **trace the steps taken by the attacker on the compromised machine** and **determine how they managed to bypass the Network Intrusion Detection System (NIDS)**.
-  "...identify the specific malware family employed in the attack and its characteristics."
- "...identify and mitigate any traces or footprints left by the attacker."

## **3. Plan**

- I snapshot my VM before doing any changes
- I first checked with my downloaded VM which was Kali Linux and I realised that I need to setup a VMnet Host-Only to isolate the VM
- I've also checked if I have the volatility tool installed which I don't have then I proceed to install it as instructed from https://github.com/volatilityfoundation/volatility3

## **4. Steps**

**Question 1: What is the name of the suspicious process?**

1. I used the command `vol -f MemoryDump.mem windows.pslist.PsList` as earlier I used `vol -f MemoryDump.mem windows.info` to confirm if it is a windows memory file.
2. Looking through the process I found a process called `oneetx.exe`, I googled it and found that it is a malicious program.
![](Attachments/Question%201.png)

Answer: oneetx.exe

**Question 2: What is the child process name of the suspicious process?**

1. Using the command `vol -f MemoryDump.mem windows.pstree.PsTree`, I looked at the process again and found the child process to be `rundll32.exe`.
![](Attachments/Question%202.png)

Answer: rundll32.exe

**Question 3: What is the memory protection applied to the suspicious process memory region?**

1. I used the command `vol -f MemoryDump.mem windows.vadinfo.VadInfo --pid 5896` to look for process memory ranges as I learned they contain memory protection attributes.
2. I noticed there are VAD and VADS. I looked up online to find the difference and learned that VAD is a kernel data while VADS is a specific node from VAD.
3. I found under `oneetx.exe` that the VADS is set to `PAGE_EXECUTE_READWRITE`.
![](Attachments/Question%203.png)

Answer: PAGE_EXECUTE_READWRITE

**Question 4: What is the name of the process responsible for the VPN connection?**

1. Using the command `vol -f MemoryDump.mem windows.pslist.PsList` again I looked for any process related to VPN and I found `tun2sock.exe`
![](Attachments/Question%204.1.png)
2. Then I used `vol -f MemoryDump.mem windows.pstree.PsTree` to see in further details. I found that it is associated with outline.exe and googling it shows that it is a VPN.
![](Attachments/Question%204.2.png)

Answer: outline.exe

**Question 5: What is the attacker's IP address?**

1. Using the command `vol -f MemoryDump.mem windows.netscan.NetScan` to find network objects in the memory. I noticed that the suspicious process has an IP address `77.91.124.20`.
![](Attachments/Question%205.png)

Answer: `77.91.124.20`

**Question 6: What is the full URL of the PHP file that the attacker visited?**

1. Using Question 5 answer process number `5896`, I proceeded to do a memory dump with memory command `vol -f MemoryDump.mem windows.memmap --pid 5896 --dump` (referred from this source: https://medium.com/@brsdncr/cyber-tool-usage-practice-volatility3-a078ac5f4f93). What it is basically is that it prints out the memory but with `--dump` it will print out the content of the memory.
2. Once is dumped, I used the command `strings pid.5896.dmp | grep ^http | grep .php` to find the readable string that starts with `http` and containing `.php`. From there I found the PHP URL the attacker visited.
![](Attachments/Question%206.png)

Answer: `http://77.91.124.20/store/games/index.php`

**Question 7: What is the full path of the malicious executable?**

1. Since the suspicious process is `oneetx.exe`. I used the command `strings pid.5896.dmp | grep oneetx.exe`.
![](Attachments/Question%207.png)

Answer: C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe
## **5. Answers**

Question 1: oneetx.exe

Question 2: rundll32.exe

Question 3: PAGE_EXECUTE_READWRITE

Question 4: outline.exe

Question 5: `77.91.124.20`

Question 6: `http://77.91.124.20/store/games/index.php`

Question 7: C:\Users\Tammam\AppData\Local\Temp\c3912af058\oneetx.exe

## **6. Lessons Learned**

- Take a break and your time as sometimes I could miss important details
- Research/investigation is key
