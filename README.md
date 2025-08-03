<p align="center">
<img src="https://logos-world.net/wp-content/uploads/2022/11/Splunk-Emblem.png" alt="Splunk Logo" width="190"/>


# Investigating Backdoor Activity Across Suspected Hosts Using Splunk

## Platform and Tools Used
- TryHackMe
- Splunk
- Virtual Machine


## Scenario Details
A SOC Analyst has observed some anomalous behaviours in the logs of a few windows machines. It looks like the adversary has access to some of these machines and successfully created some backdoor. My task as a SOC Analyst is to examine the logs and identify the anomalies. All relevant logs are in the **index="main"**.

## Objective
In this investigation, I will investigate and confirm potential indicators of compromise across suspected hosts. To do this, I will raise and methodically answer a series of key questions that guide each step of the process. This approach will help uncover any malicious activity, such as unauthorized account creation, suspicious command executions, or registry changes. The goal is to demonstrate how a structured analysis using tools like Splunk can be used to detect and respond to backdoor activity within a compromised environment.

---
#### Question 1: Do the logs show that a new username was created?
To find the answer, I first checked the "Interesting Fields" pane, where I noticed that the EventID field was present. This indicates that logging was performed by Windows and/or Sysmon from the Sysinternals suite, as both use the EventID field to assign ID numbers corresponding to specific types of events. In this case, we're interested in events where a user account was created, which is recorded as **Event ID 4720**. I then ran the following search to locate these events: **index="main" EventID="4720"**

![Screenshot 2025-05-22 013046](https://github.com/user-attachments/assets/6db7cba7-2df6-4f6f-99b3-a3890e037118)![Screenshot 2025-05-22 013239](https://github.com/user-attachments/assets/ddad888c-82f3-4a23-a76d-1dded90ff70d)


As shown in the screenshot, only one event was returned, indicating that the account named **James** created the backdoor. It also reveals the name of the new account created: **A1berto**. It's important to note the spelling—the letter "L" is replaced with the number "1", likely as an attempt to mimic the name of a legitimate user and avoid detection.
___
#### Question 2: Is there an updated registry key? If so, what is the full path of that registry key? 
Just as we identified the event logged when a new user was created, we can also search for events related to registry updates. The relevant **Event ID** for this is **13**. To investigate further, I included the suspicious username in the search and ran the following query: **index="main" EventID="13" a1berto**

![Screenshot 2025-05-22 025356](https://github.com/user-attachments/assets/b85145f3-98f4-4bd6-8600-887d2912522f)
![Screenshot 2025-05-22 025442](https://github.com/user-attachments/assets/d1a77b6f-8635-4b9a-ab9b-3961d0c4a700)

As shown in the screenshot, only one event was returned, and it contains the registry path we're looking for, along with the suspicious username: **HKLM\SAM\SAM\Domains\Account\Users\Names\A1berto**.
___
### Question 3: According to the logs, is there a legitimate user the adversary was trying to impersonate?
To find the answer, I went to the **Selected Fields** pane and selected **User** to view all the users listed in the system.
 <img src="https://github.com/user-attachments/assets/fcdc3f77-51bf-49b4-8a62-b6f5c48e09fe" alt="Screenshot" width="800"/>

As shown in the screenshot, there were four users listed in the logs. One user stood out: **Cybertees\Alberto**. This suggests that the adversary, using the username "A1berto", was attempting to impersonate the legitimate user **Alberto**.
___
### Question 4: What command is used to add a backdoor user from a remote computer?
As with the previous step, I navigated to the **Selected Fields** pane and selected **CommandLine** to review all logged commands. My goal was to identify any command containing the suspicious username.
![Screenshot 2025-05-22 165538](https://github.com/user-attachments/assets/a55fa89d-6187-49b8-8a01-d93c5bf6ecd9)
As shown in the screenshot, there was one command containing the suspicious username: **"C:\Windows\System32\Wbem\WMIC.exe" /node:WORKSTATION6 process call create "net user /add A1berto paw0rd1"** This is most likely the command used to add the backdoor user.
___
### Question 5: What is the name of the infected host where suspicious PowerShell commands were executed?
The infected host can be identified using the information from the previous screenshot, which shows the **Hostname:James.browne**.
___
### Question 6: Is PowerShell logging enabled on this device? If so, how many events were recorded related to the malicious PowerShell execution?
To find the answer, I relied on Event IDs. When PowerShell logging is enabled, Windows logs all PowerShell commands using Event ID 4103. By searching for events with this ID (**index="main" EventID="4103"**), we can determine whether PowerShell logging is enabled and how many related events were recorded.
![Screenshot 2025-05-22 191231](https://github.com/user-attachments/assets/fff6137b-1077-4bd2-ba15-e47723ea554c)
As shown in the screenshot, **79** instances of malicious PowerShell activity were detected.
___

## Conclusion 

This investigation effectively demonstrates how Splunk can be leveraged to detect and analyze malicious activities across multiple hosts. By examining Windows event logs, the analysis identified the creation of a backdoor user account (**A1berto**), registry modifications, and the execution of suspicious PowerShell commands on the host **James.browne**. Notably, PowerShell logging was enabled, capturing 79 events related to the malicious activity. The adversary's attempt to impersonate the legitimate user **Alberto** underscores the importance of vigilant monitoring and analysis. Utilizing Splunk's capabilities allowed for a comprehensive understanding of the attack vector and reinforced the critical role of proactive log analysis in cybersecurity.

