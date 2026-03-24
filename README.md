# Threat-Hunting-with-Splunk

Before starting a hunt for threats on your network, we must first learn about the operating environment. Here is a quick lab on how to do so with Splunk.
-Learning about your operating environment: 
  - Open Splunk and in the Search & Reporting app type the following SPL to view data sourcetypes, sorting them by the total count number.
  - `| metadata type=sourcetypes index="Your Data Index" | sort - totalCount  | convert ctime(*Time) `
  - A window like the below should appear. (click to expand)
  - ![s1](https://github.com/user-attachments/assets/baae1944-1912-4abe-8859-6cdd0fa84e16)
  - What can we learn about our environement based upon what we have displayed?
    - We see that Amazon Web Service (AWS) is present which likley means some or all of our enviornment is running offprem (cloud based). We also see "WinEventLog" which likley means our environment is running a configuration of Windows.
    - ![s1 1](https://github.com/user-attachments/assets/88911c1a-cb9d-431e-95d6-cb5d81a48945)
-   To confirm our assumption that our enviornment is running a configuration of Windows, we can select a single host from the network to see what it is running. If it is running any Windows services then our assumption is correct. We can use the SPL below to search our host looking for the source type Windows Event Logs.
    - ` index="Your Data Index" host="Your Single Host" sourcetype="WinEventLog*"`
<br>
-  As our results come back we see that we have quite a few results were the sourcetype is WinEventLog. It is more than safe to assume that our enviornment is running a configuration of Windows. 

![S3](https://github.com/user-attachments/assets/877a7d61-d030-4989-96a7-50e0ae820bf0)





<br><br><br>
