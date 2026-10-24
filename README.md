#  SOC Level 1 Investigation Report: Joomla Web Server Compromise
---
##  Incident Metadata 

| Field | Value |
| :--- | :--- |
| Incident ID | INC-2026-10-001 |
| Analyst Name |<span style="color:blue;">Husam Abdulkareem Mohammed Abdullah (SOC Analyst L1)</span> |
| Date of Investigation | 2026-10-20 |
| Alert Classification | True Positive (TP) |
| Incident Severity | Critical |
| Incident Status | Closed (Remediation Escalated to IT Team) |
| Affected Asset | Web Server (192.168.250.70) |
---
###  Analyst Notes 
Why was this classified as a True Positive? 
This alert was confirmed as a True Positive because further investigation proved that the attack was not just a harmless probe, but a successful full compromise. Evidence includes:
1. Successful SQL Injection (HTTP 500 errors).
2. Deployment of a Web Shell (agent.php).
3. Proof of Remote Code Execution (Sysmon logs showing cmd.exe).
4. Successful download of a malicious payload (HTTP 200).00:41
---
##  1. Executive Summary 
* <span style="color:green;">A full system compromise was detected on a Windows-based Joomla web server (192.168.250.70). The attacker utilized an automated vulnerability scanner (Acunetix) to identify a SQL Injection flaw within the Joomla search component. After successful exploitation, the attacker deployed a Web Shell (agent.php) to establish Remote Code Execution (RCE). Finally, the attacker used the compromised server to download a Remote Access Trojan (RAT) known as "Poison Ivy" from a Command & Control (C2) server, granting them persistent, full control over the victim machine. </span>
---
### Incident Details
* Environment: Splunk Enterprise (BOTS v1 Dataset)
* Victim Server (IP): 192.168.250.70 (Hostname: we1149srv)
* Attacker IP: 40.80.148.42
* C2 Server IP: 23.22.63.114
* C2 Domain: prankglassinebracket.jumpingcrab.com [The Evidence](./images/IMG_20261016_024459.jpg)

---

##  2. Cyber Kill Chain Analysis

### Phase 1: Reconnaissance
[See to Picture That Alerts Triage In Surictata](./images/IMG_AlertsTriage.jpg)
The attacker used an automated web vulnerability scanner. Suricata alerts confirmed the use of Acunetix to probe the web server for known vulnerabilities.


### Phase 2: Initial Access (SQL Injection) 
[Press to see  ](./images/IMG_Initial%20Access.jpg)
The attacker targeted the Joomla search component (/joomla/index.php/component/search/). 
* Evidence: Over 10,000 HTTP POST requests were sent to this endpoint.
* Success Indicator: The server returned HTTP 500 (Internal Server Error) over 963 times. This indicates that the malicious SQL payloads successfully disrupted the backend database queries, confirming a SQL Injection vulnerability.


### Phase 3: Execution & Persistence (Web Shell)
[Web Shell Detection ](./images/IMG_Web%20Shell%20Detection.jpg)
After gaining initial access, the attacker uploaded a malicious PHP file named agent.php to the server.
* Evidence: HTTP logs show continuous GET/POST requests to /joomla/agent.php.
* Behavior: The attacker used spoofed HTTP User-Agents (e.g., Opera, old Firefox versions) to bypass detection while interacting with the Web Shell.


### Phase 4: Command & Control (C2) 
[Press to see](./images/IMG_C2%20Detection.jpg)
The attacker established a connection to an external server to download a payload.
* Evidence: The victim server connected to prankglassinebracket.jumpingcrab.com (23.22.63.114) and requested a file named /poisonivy-is-coming-for-you-batman.jpeg.
* Analysis: The file has a .jpeg extension, but it is actually a malicious executable (Poison Ivy RAT). The server returned HTTP 200, meaning the download was successful.


### Phase 5: Actions on Objectives (Remote Code Execution) 
[Press to see](./images/IMG_Remote%20Code%20Execution.jpg)
Sysmon logs provided the definitive proof of system compromise.
* Evidence: The web server process (C:\Program Files (x86)\PHP\v5.5\php-cgi.exe) spawned a command prompt (cmd.exe).
* Commands Executed:
* cmd.exe /c "dir" and ls (File discovery)
* cmd.exe /c "ifconfig" (Network discovery)
* cmd.exe /c "3791.exe" (Execution of the downloaded RAT payload)

---
## 5. MITRE ATT&CK Mapping

To standardize the investigation, the observed adversarial behaviors were mapped to the MITRE ATT&CK framework. This mapping helps in understanding the adversary's tactics, techniques, and procedures (TTPs) and assists in building future detection rules.

| Tactic  | Technique ID | Technique Name  | Evidence from Investigation|
| :--- | :--- | :--- | :--- |
| Initial Access | T1190 | Exploit Public-Facing Application | The attacker exploited a SQL Injection vulnerability in the Joomla search component to gain initial access.[Press to See](./images/IMG_Initial%20Access.jpg) |
| Persistence | T1505.003 | Web Shell | The attacker uploaded a PHP Web Shell (agent.php) to maintain persistent access and execute commands .[Press to See](./images/IMG_20261018_230216.jpg) |
| Execution | T1059.003 | Windows Command Shell | Sysmon logs proved that php-cgi.exe spawned cmd.exe to execute commands like dir, ifconfig, and 3791.exe.[Press to See](./images/IMG_20261016_014919.jpg) |
| Command and Control | T1071.001 | Web Protocols | The victim server communicated with the C2 server (23.22.63.114) over HTTP to download payloads and receive commands. [Press to See](./images/IMG_C2%20Detection.jpg)|
| Command and Control | T1105 | Ingress Tool Transfer | The server downloaded a malicious file disguised as an image (/poisonivy-is-coming-for-you-batman.jpeg) from the C2 server. [Press to See](./images/IMG_Command&Control(C2).jpg) |
---
### Analysis of the Mapping 
* T1190 (Exploit Public-Facing Application): This confirms that the entry point was a web application vulnerability (SQLi), not a phishing attack or credential theft.
* T1505.003 (Web Shell): This is a critical finding. It shows the attacker's intent to maintain long-term access. Detecting the creation of .php files in web directories should be a high-priority alert.
* T1059.003 (Windows Command Shell): This is the most actionable detection point. Any web server process (php-cgi.exe, w3wp.exe) spawning cmd.exe is a severe anomaly that should trigger an immediate SIEM alert.
* S0012 (PoisonIvy): Identifying the specific malware family helps the threat intelligence team track the threat actor's infrastructure and potential motives.

---

##  4. Recommendations & Mitigation

1. Containment: Immediately isolate the server 192.168.250.70 from the network to prevent lateral movement.
2. Eradication: Remove the Web Shell (agent.php), the downloaded payload (.jpeg), and the executable (3791.exe). Rebuild the server from a trusted backup.
3. Network Blocking: Block the Attacker IP (40.80.148.42), C2 IP (23.22.63.114), and C2 Domain at the firewall and DNS levels.
4. Patching: Update Joomla to the latest version and upgrade the severely outdated PHP version (v5.5).
5. Detection Engineering: Create a SIEM alert to trigger whenever php-cgi.exe or w3wp.exe attempts to spawn cmd.exe or powershell.exe.