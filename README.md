# Threat-Hunting-with-Splunk

Before starting to hunt for threats on your network, you must first learn about the operating environment. You can follow a similar process I used in Splunk using this lab write up!

  - The first step in this process is to open Splunk and in the Search & Reporting app type the following SPL to view data sourcetypes, sorting them by the total count number.
  - `| metadata type=sourcetypes index="My Data Index" | sort - totalCount  | convert ctime(*Time) `
  - A window like the below will appear. (click to expand)
![s1](https://github.com/user-attachments/assets/baae1944-1912-4abe-8859-6cdd0fa84e16)
  - What did I learn about my network environement based upon what is displayed?
![s1 1](https://github.com/user-attachments/assets/88911c1a-cb9d-431e-95d6-cb5d81a48945)
    - I see that Amazon Web Service (AWS) is present which likley means some or all of the enviornment is running offprem (cloud based). I also see "WinEventLog" which likley means my environment is running a configuration of Windows.
-   To confirm my assumption that the enviornment is running a configuration of Windows, I select a single host from the network to see what it is running. If it is running any Windows services then my assumption is correct. I use the SPL below to search the host looking for the source type Windows Event Logs.
    - ` index="My Data Index" host="My Single Host" sourcetype="WinEventLog*"`
- As my results come back I see that I have quite a few results were the sourcetype is WinEventLog. It is more than safe to assume that my enviornment is running a configuration of Windows. 

![S3](https://github.com/user-attachments/assets/877a7d61-d030-4989-96a7-50e0ae820bf0)
<br><br>
Now that I have a pretty good understanding of my enviornment lets start threat hunting!
 - I will start my search by looking at some Windows System Monitor logs (Windows Sysmon). This is a good place to start looking for threats because it contains logs on system activity, network connections, and changes in the file system. A perfect place for potential threats to be discovered! To do this I type in the SPL syntax below.
   - `index="My Data Index" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"`
 - A menu on the left appears listing various fields. Some of the fields listed include process_name, Image, and file_path.
 - Let's focus on just one of our user hosts on the network and drill down into their network logins. Network logins appear as Event ID3 in the Sysmon log.
   - `index=botsv4 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" host="My Specific User Host" EventID=3`
 - I review the "Image" process field in the Selected Fields column and find something noteworthy!
<img width="1896" height="796" alt="image" src="https://github.com/user-attachments/assets/2894612b-d498-4392-a064-e749b9f2aae1" />
 - One of the top values suggests that this user has been using a Tor browser. Tor browsers are particularly forbidden on corporate networks and this should be reported to the information security team.
<br><br>
After finding one threat on the system, I now turn my efforts to try and find another! In the Search & Reporting app I search a host and the Sysmon log for logs with Event ID 1. Event ID 1 triggers whenever Windows starts a new process. This could be a great place to spot new malicious activity. I also search for the process name “powershell.exe.” My SPL syntax includes some stats and a count so I can easily review the data.

- `index=botsv4 sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=1 process_name="powershell.exe" host="TheHostIamSearching"
| stats count by host user CommandLine
| sort - count`

I select an event from the results that includes an encoded powershell command.
<br><br>
![l2 1](https://github.com/user-attachments/assets/aa9a8965-33ad-491d-aad1-39078e5e20bd)
<br>
- It looks like the command is encoded using Base64. I pull up a new web browser window and navigate over to CyberChef which is an open-source app that allows me to decode the command string.
- ![l2 4](https://github.com/user-attachments/assets/ae38f332-ad67-4760-b2d3-003661c8a010)
- <br>
The encoded command is used to download and immediately run a script from the internet which is likely malicious. This technique is used to execute code without saving a file to the disk to avoid detection since the code will run in memory. 
- IEX: tells PowerShell to take a string of text and run it as if it were a command.
- New-Object Net.WebClient: Creates a temporary web browser-like tool.
- DownloadString(http://x.x.x.x): Fetches the text content of the script located at that specific IP address.
- invoke-passkey -unlock: This executes a specific function (presumably defined inside the downloaded script) with an "unlock" parameter.
In summary this is a very suspicious and should be reported to the information security team! 






<br><br><br>
