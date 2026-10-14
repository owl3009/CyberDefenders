# **BlackEnergy Lab**

## **Tools**
Platform: CyberDefenders

Category: Endpoint Forensics

Difficulty: Medium

Tools: Volatility


## **Description**
Develop practical skills in Windows memory forensics using Volatility by detecting malware indicators, analyzing suspicious processes, and identifying code injection and unauthorized DLLs in a compromised system.

## **1. Goal**

A multinational corporation has suffered a cyber attack, resulting in the theft of sensitive data. The attack employed a previously unseen variant of the BlackEnergy v2 malware. The company's security team has obtained a memory dump from the infected machine and is seeking your expertise as a SOC analyst to analyze the dump in order to understand the scope and impact of the attack.

## **2. Key Clues**

- "A multinational corporation has suffered a cyber attack, **resulting in the theft of sensitive data.**"
- "The attack employed a previously **unseen variant of the BlackEnergy v2 malware**"
- "The company's security team has obtained **a memory dump from the infected machine**..."

## **3. Plan**

- I first checked on .raw format and learned that is a unprocessed data from a digital camera. Reason being is I am not familiar with that format.
- From there I began using volatility to investigate the issue, as is only one file.

## **4. Steps**

**Question 1: Which volatility profile would be best for this machine?**

1. Using the command `vol -f CYBERDEF-567078-20230213-171333.raw windows.info`, I found the it has the NTBuild lab which is "2600.xpsp.080413-2111" so through googling I found that it is a Window XP profile (https://betawiki.net/wiki/Windows_XP_build_2600.5512_(xpsp.080413-2111)). The Windows profile is Windows XP Service Pack 3.
2. Since I am using Volatility 3, that means I no longer have the profiles so I used an online example of what the profile would look like (https://github.com/volatilityfoundation/volatility/wiki/2.6-win-profiles)
![](Attachments/Question%201.png)

Answer: WinXPSP2x86

**Question 2: How many processes were running when the image was acquired?**

1. Using the command `vol -f CYBERDEF-567078-20230213-171333.raw windows.pslist.PsList`, I found a list of processes.
2. First I isolated the ones that does not have indication of `ExitTime` which is 6 terminated processes.
3. I then used `vol -f CYBERDEF-567078-20230213-171333.raw windows.pslist.PsList | wc -l` to find the amount of lines contained in the command. Finally I minus the unnecessary lines + `ExitTime` lines.
4. Answer = 29 (4+6) = 19
![](Attachments/Question%202.png)

Answer: 19

**Question 3: What is the process ID of `cmd.exe`?**

1. Using the previous command, I identified the PID.
![](Attachments/Question%203.png)

Answer: 1960

**Question 4: What is the name of the most suspicious process?**

1. The most suspicious process is `rootkit.exe` as it is a software designed to give unauthorised root access.
![](Attachments/Question%204.png)

Answer: rootkit.exe

**Question 5: Which process shows the highest likelihood of code injection?**

1. Using the previous question screenshot, I did some research on Windows process that are most vulnerable to code injection and found that `svchost.exe` is the most vulnerable. The reason is because is concealed elevated privilege.
![](Attachments/Question%205.png)

Answer: svchost.exe

**Question 6: There is an odd file referenced in the recent process. Provide the full path of that file.**

1. After thorough research my solution for this question is first I use the command `vol -f CYBERDEF-567078-20230213-171333.raw windows.malfind` which helps find potential injected codes. I identified like the previous process it contains the `svchost.exe`
![](Attachments/Question%206.1.png)
2. I then created a dump with the command `vol -f CYBERDEF-567078-20230213-171333.raw windows.malfind --pid 880 --dump`.
3. Finally I used the strings command to turn it into a readable file. The command is `strings pid.880.vad.0x980000-0x988fff.dmp`
![](Attachments/Question%206.2.png)

Answer: C:\WINDOWS\system32\drivers\str.sys

**Question 7: What is the name of the injected DLL file loaded from the recent process?**

1. Using the command `vol -f CYBERDEF-567078-20230213-171333.raw windows.ldrmodules.LdrModules --pid 880` I listed the loaded modules.
2. Since rootkits are often not mapped, I found that `msxml3r.dll` is the only one that isn't mapped.
![](Attachments/Question%207.png)

Answer: msxml3r.dll

**Question 8: What is the base address of the injected DLL?**

1. Using the same command as the previous command, the first `False` `MappedPath` is N/A so that would be the base address.
![](Attachments/Question%208.png)

Answer: 0x980000
## **5. Answers**

Question 1: WinXPSP2x86

Question 2: 19

Question 3: 1960

Question 4: rootkit.exe

Question 5: svchost.exe

Question 6: C:\WINDOWS\system32\drivers\str.sys

Question 7: msxml3r.dll

Question 8: 0x980000

## **6. Lessons Learned**

- Previous question can be a hint to the question your working on
- Always look through the available modules provided