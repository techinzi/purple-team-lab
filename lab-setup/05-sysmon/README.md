# Sysmon — Installation & Configuration Guide

## Overview

System Monitor (Sysmon) is a Windows system service and device driver
developed by Microsoft Sysinternals that monitors and logs system activity
to the Windows Event Log. It provides detailed information about process
creation, network connections, file creation, registry modifications, and
many other system events that standard Windows logging does not capture.

### Role in Purple Team Lab

Without Sysmon, Windows endpoint visibility is severely limited:

| Activity | Without Sysmon | With Sysmon |
|---|---|---|
| Process execution | Basic process name only | Full command line, parent process, hashes |
| Network connections | Nothing | Remote IP, port, process that connected |
| File creation | Nothing | Full file path, process that created it |
| Registry changes | Nothing | Key, value, old and new data |
| Driver loading | Nothing | Driver path and signature status |
| Process injection | Nothing | Source and target process details |

In the Purple Team lab, Sysmon is the primary source of endpoint telemetry.
Every adversary emulation exercise generates Sysmon events that are forwarded
to Wazuh for detection validation.

---

## Part 1 — Download Sysmon

### Step 1 — Download Sysmon from Microsoft Sysinternals

Open browser on WIN11-CLIENT01. Navigate to the official [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) page.

Click **"Download Sysmon"**.

![Image](/images/lab-setup/sysmon/01-download.png)

### Step 2 — Extract Sysmon

Open PowerShell as Administrator and extract to `C:\Tools\Sysmon`:

```powershell
# Create Sysmon directory
New-Item -ItemType Directory -Path "C:\Tools\Sysmon" -Force

# Extract Sysmon zip
Expand-Archive -Path "$env:USERPROFILE\Downloads\Sysmon.zip" -DestinationPath "C:\Tools\Sysmon" -Force

# Verify extracted files
Get-ChildItem C:\Tools\Sysmon
```

![Image](/images/lab-setup/sysmon/02-download.png)

---

## Part 2 — Download Sysmon Configuration

Sysmon without a configuration file provides minimal useful telemetry.
A configuration file defines which events to capture, which to exclude,
and what filters to apply. Always install Sysmon with a configuration file.

### Step 1 — Download SwiftOnSecurity Sysmon Configuration

The SwiftOnSecurity Sysmon configuration is the most widely used
community configuration. It provides comprehensive coverage of attacker
techniques while filtering out known-good noise.

```powershell
# Download SwiftOnSecurity config
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\Sysmon\sysmonconfig.xml"

# Verify download:
Get-ChildItem C:\Tools\Sysmon
```

![Image](/images/lab-setup/sysmon/03-download.png)

> **Why SwiftOnSecurity configuration?**
>
> SwiftOnSecurity is the industry standard starting point. It covers the
> most important ATT&CK techniques while aggressively filtering legitimate
> Windows activity to reduce noise.

---

## Part 3 — Install Sysmon

### Step 1 — Accept EULA and Install Sysmon

Run the following command in PowerShell as Administrator:

```powershell
C:\Tools\Sysmon\Sysmon64.exe -accepteula -i C:\Tools\Sysmon\sysmonconfig.xml
```

| Flag | Meaning |
|---|---|
| `-accepteula` | Accepts the End User License Agreement silently |
| `-i` | Install Sysmon with the specified configuration file |

![Image](/images/lab-setup/sysmon/04-installation.png)

---

## Part 4 — Verify Sysmon Installation

### Step 1 — Verify Sysmon Service is Running

```powershell
Get-Service Sysmon64
```

![Image](/images/lab-setup/sysmon/05-installation.png)

### Step 2 — Verify Sysmon Driver is Running

```powershell
Get-Service SysmonDrv
```

![Image](/images/lab-setup/sysmon/06-installation.png)

### Step 3 — Verify Sysmon Configuration is Loaded

```powershell
C:\Tools\Sysmon\Sysmon64.exe -c
```

![Image](/images/lab-setup/sysmon/07-installation.png)

### Step 4 — Verify Sysmon Event Log Channel Exists

```powershell
Get-WinEvent -ListLog "Microsoft-Windows-Sysmon/Operational" | Select-Object LogName, IsEnabled, RecordCount
```

![Image](/images/lab-setup/sysmon/08-installation.png)

---

## Part 5 — Sysmon Event IDs Reference

Understanding Sysmon Event IDs is fundamental to Purple Team work.
Each Event ID corresponds to a specific type of system activity.

### Critical Event IDs for Purple Team Lab

| Event ID | Name | ATT&CK Relevance | Purple Team Use |
|---|---|---|---|
| **1** | Process Creation | T1059 Execution techniques | Most important — captures all process launches with full command line |
| **2** | File Creation Time Changed | T1070 Timestomping | Anti-forensics detection |
| **3** | Network Connection | T1071 C2 communications | Detects outbound connections from suspicious processes |
| **5** | Process Terminated | All execution techniques | Process lifecycle tracking |
| **7** | Image Loaded | T1055 Process injection | DLL loading — detects injection and LOLBin abuse |
| **8** | CreateRemoteThread | T1055 Process injection | Thread injection detection |
| **10** | ProcessAccess | T1003 Credential dumping | LSASS access — critical for detecting Mimikatz |
| **11** | FileCreate | T1105 Tool transfer | File drop detection |
| **12** | RegistryEvent (Object create/delete) | T1547 Persistence | Registry key creation |
| **13** | RegistryEvent (Value Set) | T1547 Registry run keys | Registry value modification |
| **14** | RegistryEvent (Key/Value Rename) | T1112 Modify registry | Registry rename operations |
| **15** | FileCreateStreamHash | T1564 Hidden files | Alternate data stream detection |
| **17** | PipeEvent (Pipe Created) | T1021 Lateral movement | Named pipe creation |
| **18** | PipeEvent (Pipe Connected) | T1021 Lateral movement | Named pipe connection |
| **22** | DNSEvent | T1071 DNS C2 | DNS query logging |
| **23** | FileDelete | T1070 Indicator removal | File deletion tracking |
| **25** | ProcessTampering | T1055 Process injection | Process image tampering |

> **Most Important for Purple Team:** Event IDs 1, 3, 8, 10, 11, 13, 22
>
> These seven Event IDs cover the majority of ATT&CK technique detections.

---

## Part 6 — Generate and Verify Test Events

### Step 1 — Generate a Process Creation Event (EID 1)

```powershell
# Launch notepad to generate a process creation event
Start-Process notepad.exe
Start-Sleep -Seconds 2
Stop-Process -Name notepad -Force
```

![Image](/images/lab-setup/sysmon/09-verification.png)

### Step 2 — Generate a Network Connection Event (EID 3)

```powershell
# Make a network connection to generate EID 3
Invoke-WebRequest -Uri "http://example.com" -UseBasicParsing | Out-Null
```

![Image](/images/lab-setup/sysmon/10-verification.png)

### Step 3 — Generate a Registry Event (EID 13)

```powershell
# Write a test registry value to generate EID 13
New-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "SysmonTest" -Value "C:\Tools\test.exe" -PropertyType String -Force

# Clean up test value immediately
Remove-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "SysmonTest" -Force
```

![Image](/images/lab-setup/sysmon/11-verification.png)

### Step 4 — Verify Events in PowerShell

```powershell
# Check for recent Sysmon events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20 | Select-Object TimeCreated, Id, Message | Format-Table -AutoSize
```

![Image](/images/lab-setup/sysmon/12-verification.png)

### Step 5 — Verify Specific Event ID

```powershell
# Check specifically for EID 1 (Process Creation) events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Where-Object { $_.Id -eq 1 -and $_.Message -match "notepad.exe" -and $_.Message -match "powershell.exe" } | Format-List TimeCreated, Message
```

![Image](/images/lab-setup/sysmon/13-verification.png)

---

## Part 7 — Verify Events in Event Viewer

### Step 1 — Open Event Viewer

Press `Win + R` > Type: `eventvwr.msc` > Press `Enter`


### Step 2 — Navigate to Sysmon Log

`Event Viewer` > `Applications and Services Logs` > `Microsoft` > `Windows` > `Sysmon` > `Operational`

### Step 3 — View Sysmon Events

The Operational log shows all Sysmon events in chronological order.

Click any event to see full details in the bottom panel:

![Image](/images/lab-setup/sysmon/14-verification.png)

> **Note:** Sysmon events display two user accounts.The System user identifies
> the account under which Sysmon writes events (typically `SYSTEM`),
> while the EventData user identifies the account that actually performed the
> activity being monitored (for example, `WIN11-CLIENT01\labuser` launching `notepad.exe`).

### Step 4 — Filter Events by Event ID

Use `Filter Current Log...` to filter for specific Event IDs in Event Viewer.

![Image](/images/lab-setup/sysmon/15-verification.png)

> **Note:** The User field in `Filter Current Log` filters on the account that
> wrote the event to the Windows Event Log, not the account that performed
> the monitored activity.
>
> To identify events generated by a specific user, use PowerShell to filter the event message contents.

---

## Part 8 — Update Sysmon Configuration

### Step 1 — Update Configuration Without Reinstalling

When a new version of the SwiftOnSecurity config is released or we
want to apply a different configuration:

```powershell
# Download latest config
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\Sysmon\sysmonconfig-new.xml"

# Apply new config without reinstalling
C:\Tools\Sysmon\Sysmon64.exe -c C:\Tools\Sysmon\sysmonconfig-new.xml

# Verify new config is active
C:\Tools\Sysmon\Sysmon64.exe -c
```

---

## Part 9 — Remove or Reinstall Sysmon

If required, we can remove or reinstall sysmon as below.

### Step 1 — Uninstall Sysmon

```powershell
# Uninstall Sysmon (run as Administrator)
C:\Tools\Sysmon\Sysmon64.exe -u
```

### Step 2 — Reinstall Sysmon

After uninstalling, reinstall with the same command used earlier:

```powershell
C:\Tools\Sysmon\Sysmon64.exe -accepteula -i C:\Tools\Sysmon\sysmonconfig.xml
```

---

## Part 10 — Take Snapshot

### Step 1 — Take Snapshot After Sysmon Installation

Lock the screen before taking snapshot: `Win + L`

In VMware Workstation: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `Sysmon Installed` |
| Description | `Sysmon installed with SwiftOnSecurity config. No Wazuh agent yet.` |

Click **"Take Snapshot"**.

![Image](/images/lab-setup/sysmon/16-verification.png)

---
