# Cloud-Based-SOC-Lab-with-Wazuh-and-Sysmon

## 🚀 Project Overview
This project demonstrates the implementation of a cloud-based Security Operations Center (SOC) environment. I deployed a Wazuh Manager on an Ubuntu cloud instance (cPouta) to monitor a local Windows 10 victim machine. By integrating Sysmon, I achieved high-fidelity telemetry that allowed for the detection of advanced attack techniques, including network reconnaissance and authentication brute-forcing.

---

## 🏗️ Architecture & Tools
* **SIEM Platform:** [Wazuh 4.9](https://wazuh.com/)
* **Cloud Infrastructure:** cPouta Cloud (Ubuntu 22.04 LTS)
* **Endpoint Telemetry:** Sysmon with Olaf Hartong's Modular Configuration
* **Victim OS:** Windows 10 (Virtual Machine)
* **Attacker OS:** Kali Linux (Virtual Machine)



---

## 🛠️ Step-by-Step Lab Setup

### 1. Manager Deployment (Cloud)
1. **Provision Instance:** Deploy an Ubuntu 22.04 instance on cPouta.
2. **Install Wazuh:** Execute the assisted installation script:
   ```bash
   curl -sO [https://packages.wazuh.com/4.9/wazuh-install.sh](https://packages.wazuh.com/4.9/wazuh-install.sh) && sudo bash ./wazuh-install.sh -a

2. Victim Configuration (Windows 10, powershell)

Install Wazuh Agent

Download the .msi and enroll it using the Manager's IP:

```bash
.\wazuh-agent-4.9.msi /q WAZUH_MANAGER="YOUR_MANAGER_IP"
```
Deploy Sysmon

Install Sysmon with a custom configuration file:
```bash
.\Sysmon64.exe -i sysmonconfig.xml -accepteula
```
Enable Security Auditing

Ensure failed logons are recorded for SIEM ingestion
```bash
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```
Agent Integration

Link Sysmon to Wazuh by adding this to ossec.conf:
```bash
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

3. Attacker Configuration (Kali Linux)
Ensure the Kali VM is on the same NAT network as the Windows VM to enable connectivity.

Temporarily disable the Windows Firewall to allow remote scanning:

```bash
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

⚔️ Simulated Attacks & Detections
Phase 1: Network Reconnaissance
Technique: Stealth SYN scan from Kali Linux.

Attack Command:
```bash
nmap -sS -p 1-1000 <My_Victim_IP>
```
Detection: Triggered Rule 92031: Discovery activity executed in the Wazuh dashboard.

Phase 2: Authentication Brute Force
During testing, standard rockyou.txt dictionary attacks were filtered by network security controls. I successfully pivoted to a PowerShell loop to simulate a high-velocity brute-force attempt.

Attack Command:
```bash
1..50 | ForEach-Object { net use \\localhost /user:fakeuser "wrongpassword123" }
```
Detection: This activity generated multiple Event ID 4625 (Failed Logon) alerts, leading to Rule 60122: Multiple Windows logon failures.

🔧 SIEM Tuning & Optimization
I identified Rule 92213 as a source of false positives due to legitimate PowerShell policy checks (__PSScriptPolicyTest). I implemented a custom exclusion rule in local_rules.xml to tune the SIEM:

```bash
<rule id="100001" level="0">
  <if_sid>92213</if_sid>
  <field name="win.eventdata.targetFilename" type="pcre2">__PSScriptPolicyTest_.*\.ps1$</field>
  <description>Ignore legitimate PowerShell Script Policy Test files</description>
</rule>
```
## Next Steps
I plan to deploy a public honeypot to capture and analyze live threat actor behavior in the wild
   
