# Cloud-Based-SOC-Lab-with-Wazuh-and-Sysmon
🚀 Project Overview
This project involved setting up a comprehensive Security Operations Center (SOC) home lab to simulate real-world attacks and detect them using a Security Information and Event Management (SIEM) system. The lab consists of a cloud-hosted Wazuh Manager (Ubuntu) and a local Windows 10 Victim Machine equipped with Sysmon for advanced telemetry.

The goal was to demonstrate the full defensive lifecycle: from initial reconnaissance and brute-force attempts to advanced persistence mechanisms.

🏗️ Architecture & Tools
Wazuh Manager: Hosted on cPouta Cloud (Ubuntu 22.04 LTS).

Wazuh Agent: Installed on a local Windows 10 VM (VMware).

Sysmon: Configured with a modular ruleset for high-fidelity endpoint logging.

Attacker Host: Kali Linux (Local VM) used for network-based attacks.

Tools: PowerShell, Hydra, Nmap, and Custom OSSEC Rules.

🛠️ Configuration & Implementation
1. Endpoint Telemetry (Sysmon)
To gain deep visibility into the Windows machine, Sysmon was installed with a configuration optimized for the MITRE ATT&CK framework.

Command: Sysmon64.exe -i sysmonconfig.xml.

Integration: The Wazuh agent was configured to monitor the Microsoft-Windows-Sysmon/Operational event channel in ossec.conf.

2. SIEM Rule Tuning
Custom rules were added to the Wazuh Manager to handle specific lab scenarios and reduce false positives:

Rule 92213 Tuning: Documented the identification of __PSScriptPolicyTest as a false positive and demonstrated how to create exclusion rules in local_rules.xml.

⚔️ Attack Simulations & Detections
1. Network Reconnaissance (Nmap)
Technique: Stealth SYN scan from Kali Linux targeting the victim.

Detection: Triggered Rule 92031 (Discovery activity executed) as the system attempted to respond to multiple rapid connection probes.

2. SMB Brute Force (Hydra)
Technique: Automated credential guessing using the rockyou.txt wordlist via SMB.

Detection: Triggered Rule 60122 (Multiple Windows logon failures).

Outcome: Successfully validated Active Response, resulting in a firewall block of the Attacker's IP.

3. Persistence via Scheduled Tasks
Technique: Creation of a persistent backdoor using schtasks.

Detection: Triggered Rule 92154 (Scheduled task created) after enabling the TaskScheduler/Operational log channel.

📈 Key Metrics & Results
Visibility: 100% of simulated malicious process executions were captured via Sysmon Event ID 1.

Response: Automated IP blocking reduced the duration of brute-force attacks from minutes to seconds.

Telemetry: Successfully integrated non-standard Windows logs (Sysmon/Task Scheduler) into a centralized cloud dashboard.

💡 Lessons Learned
Telemetry at the Source: Confirmed that SIEM detection is only as strong as host-based auditing (e.g., needing auditpol for logon failure logging).

SIEM Tuning: Gained experience in differentiating between legitimate administrative activity and malicious reconnaissance.

Network Isolation: Overcame virtual network isolation issues between Kali and Windows VMs to enable remote attack simulations.
