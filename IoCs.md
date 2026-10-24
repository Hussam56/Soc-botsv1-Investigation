#  <u>Indicators of Compromise (IoCs)</u>

This document contains all the Tactical Threat Intelligence (IoCs) extracted during the investigation. Security teams can use these to block the attacker and hunt for similar threats across the network.

## 1. <u>Network Indicators</u>
These IP addresses and domains are associated with the attacker's infrastructure and should be blocked.

| Indicator Type | Value | Description |
| :--- | :--- | :--- |
| IPv4 Address | 40.80.148.42 | The external IP used by the attacker for scanning and initial exploitation. |
| IPv4 Address | 23.22.63.114 | Command & Control (C2) server IP hosting the malicious payload. |
| Domain Name | prankglassinebracket.jumpingcrab.com | The DGA (Domain Generation Algorithm) domain resolving to the C2 server. |
| URI Path | /joomla/agent.php | The path of the Web Shell (Backdoor) uploaded to the server. |
| URI Path | /poisonivy-is-coming-for-you-batman.jpeg | The URL path used to download the malicious RAT payload. |

## 2. <u>Host-based Indicators</u>
These artifacts were found on the compromised Windows server (192.168.250.70).

| Indicator Type | Value | Description |
| :--- | :--- | :--- |
| File Name | agent.php | A PHP Web Shell used for Remote Code Execution (RCE). |
| File Name | poisonivy-is-coming-for-you-batman.jpeg | A malicious executable disguised as an image file (Poison Ivy RAT). |
| File Name | 3791.exe | The extracted/executed payload of the RAT. |
| Process Activity | php-cgi.exe spawning cmd.exe | Anomalous behavior indicating that the web server is executing OS commands. |
| Command Line | cmd.exe /c "3791.exe" | The exact command used to execute the malicious RAT. |