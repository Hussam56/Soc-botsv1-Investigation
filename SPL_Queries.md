
#  Splunk SPL Queries (Investigation Log)

This document contains the Splunk Processing Language (SPL) queries used to investigate the incident, categorized by the Cyber Kill Chain phases.
### 1. Reconnaissance & Alert Triage
##### Objective: Identify top security alerts and the attacking IPs.
[ See to Picture That Alerts Triage In Surictata  ](./images/IMG_AlertsTriage.jpg)
index=botsv1 sourcetype=suricata event_type=alert
|sort + _time
| table _time src_ip, dest_ip, alert.signature
|head 40

### 2. Initial Access (Web Exploitation)
##### Objective: Prove the SQL Injection by looking at HTTP status codes.
[Discovery the number of Requests ](./images/IMG_Initial%20Access.jpg)
index=botsv1 sourcetype="stream:http" src_ip=40.80.148.42 dest_ip=192.168.250.70
| stats count by status, uri_path, http_method
| sort - count

### 3. Command & Control (C2) Communication
##### Objective: Track the victim server reaching out to the external C2 server.
[Discover (C2)](./images/IMG_C2%20Detection.jpg)
index=botsv1 sourcetype="suricata" event_type=http dest_ip=23.22.63.114
| stats count by src_ip, http.hostname, http.url, http.status
[Command&Control(C2)](./images/IMG_Command&Control(C2).jpg)
 

### 4. Web Shell Detection
[Web Shell Detection ](./images/IMG_Web%20Shell%20Detection.jpg)
##### Objective: Find the requests sent to the backdoor (agent.php) and analyze User-Agents.
index=botsv1 sourcetype="stream:http" uri_path="*agent.php*"
| table _time, src_ip, http_user_agent, http_method, status
| sort _time

### 5. Remote Code Execution (Sysmon Analysis)
[Remote Code Execution](./images/IMG_Remote%20Code%20Execution.jpg)
##### Objective: Prove that the Web Server (php-cgi.exe) executed OS commands (cmd.exe).
index=botsv1 host=we1149srv sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventID=1
| search ParentImage="*php-cgi.exe*"
| table _time, ParentImage, Image, CommandLine, User
| sort _time