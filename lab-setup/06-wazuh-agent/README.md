# Wazuh Agent — Installation & Configuration Guide

## Overview

The Wazuh Agent is a lightweight service installed on monitored endpoints
that collects security telemetry and forwards it to the Wazuh Manager for
analysis and alerting.

### What the Wazuh Agent Collects

| Log Source | Events Collected | Purple Team Relevance |
|---|---|---|
| Sysmon | Process creation, network connections, registry changes | Primary telemetry source for ATT&CK technique detection |
| Windows Security Log | Logon events, account changes, privilege use | Authentication and lateral movement detection |
| Windows System Log | Service installation, driver loading | Persistence detection |
| Windows Application Log | Application crashes, errors | Malware behavior indicators |
| File Integrity Monitoring | File creation, modification, deletion | Payload delivery detection |

---

## Prerequisites

### Network Requirements

Verify the following before starting:

| Requirement | Value |
|---|---|
| WIN11-CLIENT01 IP | 192.168.48.128 |
| ubuntu-server01 IP | 192.168.48.129 |
| Wazuh Manager port | 1514 (agent communication) |
| Wazuh enrollment port | 1515 (agent registration) |

### Step 1 — Verify Services & Network Connectivity

On ubuntu-server01:

```bash
# Verify all Wazuh services are active
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat

# Verify UFW allows all Wazuh ports
sudo ufw status | grep -E '443|1514|1515|1516|9200|9300|55000'
```
> If any service not found active; enable & start the service.
>
> `sudo systemctl enable serviceName`
>
> `sudo systemctl start serviceName`
>
> If any port not allowed, allow it.

![Image](/images/lab-setup/wazuh-agent/01-pre-requisite.png)

Now, open PowerShell as Administrator on WIN11-CLIENT01:

```powershell
# Test basic connectivity to Wazuh Manager
Test-NetConnection -ComputerName 192.168.48.129 -Port 1514

# Test enrollment port
Test-NetConnection -ComputerName 192.168.48.129 -Port 1515
```

![Image](/images/lab-setup/wazuh-agent/02-pre-requisite.png)

> TcpTestSucceeded should show as **True**

---

## Part 1 — Download Wazuh Agent

### Step 1 — Download Wazuh Agent MSI

Open PowerShell as Administrator on WIN11-CLIENT01:

```powershell
# Create download directory
New-Item -ItemType Directory -Path "C:\Tools\WazuhAgent" -Force

# Download Wazuh Agent 4.9.2 MSI
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi" -OutFile "C:\Tools\WazuhAgent\wazuh-agent-4.9.2-1.msi"

# Verify download:
Get-ChildItem C:\Tools\WazuhAgent
```

![Image](/images/lab-setup/wazuh-agent/03-download.png)
---

## Part 2 — Install and Register Wazuh Agent

### Step 1 — Install Wazuh Agent with Manager Configuration

Install the agent and point it directly to the Wazuh Manager in a
single command:

```powershell
Start-Process msiexec.exe -ArgumentList "/i C:\Tools\WazuhAgent\wazuh-agent-4.9.2-1.msi /quiet WAZUH_MANAGER=192.168.48.129 WAZUH_AGENT_NAME=WIN11-CLIENT01 WAZUH_REGISTRATION_SERVER=192.168.48.129" -Wait
```

| Parameter | Value | Purpose |
|---|---|---|
| `WAZUH_MANAGER` | `192.168.48.129` | IP of Wazuh Manager to send logs to |
| `WAZUH_AGENT_NAME` | `WIN11-CLIENT01` | Name shown in Wazuh Dashboard |
| `WAZUH_REGISTRATION_SERVER` | `192.168.48.129` | IP of server to register agent with |
| `/quiet` | — | Silent installation — no GUI prompts |

![Image](/images/lab-setup/wazuh-agent/04-installation.png)

> When passing MSI properties such as `WAZUH_MANAGER`, `WAZUH_AGENT_NAME`, and
> `WAZUH_REGISTRATION_SERVER` to msiexec, do not wrap the values in single quotes.
> The quotes may be written literally into `ossec.conf`, causing the Wazuh Agent
> to use invalid configuration values and preventing successful enrollment.

### Step 2 — Verify Installation Files

```powershell
# Verify Wazuh agent installed
Test-Path "C:\Program Files (x86)\ossec-agent"

# List agent directory
Get-ChildItem "C:\Program Files (x86)\ossec-agent" | Select-Object Name, LastWriteTime
```

![Image](/images/lab-setup/wazuh-agent/05-installation.png)

---

## Part 3 — Start and Validate Agent Service

### Step 1 — Start Wazuh Agent Service

```powershell
# Start Wazuh agent service
Start-Service -Name WazuhSvc

# Set service to start automatically on boot
Set-Service -Name WazuhSvc -StartupType Automatic

# Verify agent service is running
Get-Service WazuhSvc
```
![Image](/images/lab-setup/wazuh-agent/06-installation.png)

### Step 2 — Verify Agent Registration

SSH into ubuntu-server01 and check registered agent:

```bash
sudo /var/ossec/bin/agent_control -l
```

![Image](/images/lab-setup/wazuh-agent/07-installation.png)

### Step 3 — Verify Agent in Wazuh Dashboard

On Windows 11 local host, login to Wazuh Dashboard with admin credentials: 

```
https://192.168.48.129
```

![Image](/images/lab-setup/wazuh-agent/07-installation-01.png)

Navigate to: `Left sidebar > Server management > Endpoints Summary`

![Image](/images/lab-setup/wazuh-agent/08-installation.png)

**WIN11-CLIENT01** should appear with status `Active`

![Image](/images/lab-setup/wazuh-agent/09-installation.png)

Click on **WIN11-CLIENT01** in the Endpoint Summary to view details:

![Image](/images/lab-setup/wazuh-agent/10-installation.png)

---

## Part 4 — Configure Agent to Collect Sysmon Logs

By default, the Wazuh Agent collects standard Windows Event Logs but
does not collect Sysmon events. This must be configured manually.

### Step 1 — Open Agent Configuration File

```powershell
# Open agent configuration file in Notepad
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

![Image](/images/lab-setup/wazuh-agent/11-configuration.png)

### Step 2 — Add Sysmon Log Collection

In the `ossec.conf` file, locate the `<ossec_config>` section and find
existing `<localfile>` entries.

![Image](/images/lab-setup/wazuh-agent/12-configuration.png)

![Image](/images/lab-setup/wazuh-agent/13-configuration.png)

Add the following Sysmon configuration
at the end of existing localfile entries:

```xml
<!-- Sysmon Event Log Collection -->
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Save the file: `Ctrl + S`

![Image](/images/lab-setup/wazuh-agent/14-configuration.png)

> **Why eventchannel format?**
>
> The `eventchannel` format tells the Wazuh Agent to read from the Windows
> Event Log API directly — the correct method for all modern Windows event
> logs including Sysmon.

### Step 3 — Restart Wazuh Agent to Apply Configuration

```powershell
Restart-Service WazuhSvc
Start-Sleep -Seconds 5
Get-Service WazuhSvc
```

![Image](/images/lab-setup/wazuh-agent/15-configuration.png)

---

## Part 5 — Validate Sysmon Telemetry in Wazuh

### Step 1 — Generate a Test Sysmon Alert

On **WIN11-CLIENT01**, run a test command to generate Sysmon telemetry:

```powershell
# Trigger Sysmon Event ID 11 (File Create)
Copy-Item C:\Windows\System32\notepad.exe C:\Users\labuser\evil.exe
```

![Image](/images/lab-setup/wazuh-agent/16-validation.png)

### Step 2 — Verify Sysmon Alert in Wazuh Dashboard

In Wazuh Dashboard, navigate to: `Left sidebar > Threat intelligence > Threat Hunting`

![Image](/images/lab-setup/wazuh-agent/17-validation.png)

On the landing page, we see that Wazuh Manager (ubuntu-server01) and Wazuh Agent (agent.id 001) are automatically added to filter. 

![Image](/images/lab-setup/wazuh-agent/18-validation.png)

In the search bar (DQL), filter for Sysmon alerts:

```
rule.groups : "sysmon"
```

Click **Refresh**.

Sysmon alert should appear showing creation of `evil.exe`.

![Image](/images/lab-setup/wazuh-agent/19-validation.png)

Click on the "**Magnifying Glass**" icon against generated Sysmon event to expand the full details:

![Image](/images/lab-setup/wazuh-agent/20-validation.png)

### Step 3 — Verify Alert from Ubuntu Server

SSH into ubuntu-server01 and confirm alert:

```bash
# Check alerts log for Sysmon events
sudo grep -i "evil.exe" /var/ossec/logs/alerts/alerts.log
```

![Image](/images/lab-setup/wazuh-agent/21-validation.png)

---

## Part 6 — Take Snapshot

### Step 1 — Take Snapshot After Agent Configuration

Lock the screen: `Win + L`

In VMware Workstation: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `Wazuh Agent Installed` |
| Description | `Sysmon installed. Wazuh Agent 4.9.2 installed and connected to ubuntu-server01. Sysmon log collection configured. Agent shows Active in Wazuh Dashboard.` |

Click **"Take Snapshot"**.

![Image](/images/lab-setup/wazuh-agent/22-validation.png)

---
