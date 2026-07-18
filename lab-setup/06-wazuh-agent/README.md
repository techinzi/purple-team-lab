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

## Part 6 — Wazuh Archives Index Configuration

### Why Archives are Required for Purple Team Work

Purple Team activities require access to both detection alerts and the
underlying raw telemetry generated during adversary simulation.

By default Wazuh only stores events that trigger a detection rule
in the **`wazuh-alerts-*`** index. Raw events that do not match any
rule — which includes most baseline telemetry — are discarded and
never sent to the Wazuh Indexer.

The **`wazuh-archives-*`** index stores the raw events received and decoded by
the Wazuh Manager, regardless of whether they trigger a detection rule.
Raw archived events provide the ground truth needed to:
- validate telemetry collection
- understand event structure
- inspect available fields
- determine whether a missing alert is caused by a **telemetry gap** or a **detection gap**.

The `wazuh-archives-*` index does not appear in the Wazuh Dashboard
by default because archive logging is **disabled out of the box**:

`Left sidebar > Explore > Discover`

![Image](/images/lab-setup/wazuh-agent/22-archive-index.png)

![Image](/images/lab-setup/wazuh-agent/23-archive-index.png)

> `wazuh-archives-*` index is not available.

---

### Step 1 — Enable Archives in Wazuh Manager

SSH into **UBUNTU-SERVER01**:

```bash
ssh labadmin@192.168.48.129
```

Edit the Wazuh Manager configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Find the `<global>` section and set `logall` and `logall_json` to `yes`:

![Image](/images/lab-setup/wazuh-agent/24-archive-index.png)

Save: **Ctrl + O** > **Enter** > **Ctrl + X**

> **What these settings do:**
>
> | Setting | Effect |
> |---|---|
> | `logall` | Writes all events to `/var/ossec/logs/archives/archives.log` |
> | `logall_json` | Writes all events in JSON format to `/var/ossec/logs/archives/archives.json` — required for Filebeat ingestion |

Restart Wazuh Manager:

```bash
sudo systemctl restart wazuh-manager
```

Verify archive files are being created:

```bash
sudo ls -lh /var/ossec/logs/archives/
```

![Image](/images/lab-setup/wazuh-agent/25-archive-index.png)

---

### Step 2 — Enable Filebeat to Forward Archives

Filebeat must be configured to read the archives JSON file and
forward it to the Wazuh Indexer.

Edit the Filebeat configuration:

```bash
sudo nano /etc/filebeat/filebeat.yml
```

Find the archives module section and set `enabled` to `true`:

![Image](/images/lab-setup/wazuh-agent/26-archive-index.png)

Save: **Ctrl + O** > **Enter** > **Ctrl + X**

Restart Filebeat:

```bash
sudo systemctl restart filebeat
```

Verify Filebeat is running:

```bash
sudo systemctl status filebeat | grep Active
```

Check Filebeat logs for archive ingestion:

```bash
sudo tail -20 /var/log/filebeat/filebeat
```

Look for: `INFO Harvester started for file: /var/ossec/logs/archives/archives.json`

---

### Step 3 — Create Index Pattern in Wazuh Dashboard

The `wazuh-archives-*` index pattern must be manually created in the
Dashboard before archived events become searchable.

1. Open browser on Windows 11 host: `https://192.168.48.129`

2. Navigate to: `Left sidebar > Dashboard management > Dashboards Management > Index patterns > Create index pattern`

![Image](/images/lab-setup/wazuh-agent/27-archive-index.png)

3. Enter the index pattern: `wazuh-archives-*`

4. Click **Next step**

![Image](/images/lab-setup/wazuh-agent/28-archive-index.png)

5. Select time field: `timestamp`

6. Click **Create index pattern**

![Image](/images/lab-setup/wazuh-agent/29-archive-index.png)

---

### Step 4 — Verify Archives Appear in Dashboard

Navigate to: `Left sidebar > Explore > Discover > Index selector (top left dropdown)`

![Image](/images/lab-setup/wazuh-agent/30-archive-index.png)

We should now see all raw events — including those that did not
match any Wazuh detection rule.

---

### Index Comparison

| Index | Contains | Use Case |
|---|---|---|
| `wazuh-alerts-*` | Events that matched a Wazuh rule | Alert triage, detection validation |
| `wazuh-archives-*` | All events regardless of rule match | Baseline analysis, gap identification, raw telemetry review |
| `wazuh-monitoring-*` | Wazuh agent status and health | Agent connectivity monitoring |
| `wazuh-statistics-*` | Wazuh performance metrics | Platform health monitoring |

---

### Important — Disk Space Warning

Enabling `logall_json` significantly increases disk usage on
**UBUNTU-SERVER01** because every event from every agent will now
be stored in the Wazuh Indexer — not just alerted events.

Monitor disk usage after enabling:

```bash
df -h /
```

If disk usage grows too quickly:

```bash
# Check archive index size in OpenSearch
curl -k -u admin:'YourPassword' https://localhost:9200/_cat/indices/wazuh-archives-*?v\&s=store.size:desc

# Delete old archive indices if needed (replace date)
curl -k -u admin:'YourPassword' -X DELETE https://localhost:9200/wazuh-archives-4.x-YYYY.MM.DD
```

---

## Part 7 — Take Snapshot

### Step 1 — Take *WIN11-CLIENT01* Snapshot After Agent Configuration

Lock the screen: `Win + L`

In VMware Workstation: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `Wazuh Agent Installed` |
| Description | `Sysmon installed. Wazuh Agent 4.9.2 installed and connected to ubuntu-server01. Sysmon log collection configured. Agent shows Active in Wazuh Dashboard.` |

Click **"Take Snapshot"**.

![Image](/images/lab-setup/wazuh-agent/31-snapshot.png)

### Step 2 — Take *ubuntu-server01* Snapshot After Archives Index Configuration

```bash
# Clear bash history
history -c && history -w

#Exit SSH session
exit
```

In VMware, with UBUNTU-SERVER01 VM running:

`VM menu > Snapshot > Take Snapshot`

| **Field** | **Value** |
|---|---|
| Name | `Wazuh Archives Enabled` |
| Description | `logall and logall_json enabled. Filebeat archives module enabled. wazuh-archives-* index pattern created in Dashboard.` |

---
