# Atomic Red Team — Installation & Usage Guide

## Overview

Atomic Red Team is an open-source library of small, focused adversary
technique tests mapped directly to the MITRE ATT&CK framework. Each
"atomic test" executes exactly one ATT&CK technique in a controlled,
repeatable way — making it the primary tool for technique-level
adversary emulation in Purple Team exercises.

### Role in Purple Team Lab

| Function | Description |
|---|---|
| **Adversary Emulation** | Execute real ATT&CK techniques in a controlled environment |
| **Detection Validation** | Confirm whether Wazuh alerts fire for each technique |
| **Coverage Testing** | Systematically test techniques across ATT&CK tactics |
| **Gap Identification** | Identify techniques with no detection coverage |
| **Evidence Generation** | Produce telemetry evidence for ATT&CK Navigator layers |

---

## Safety Considerations

Atomic Red Team executes real adversary techniques. Understand these
risks before running any tests:

| Risk | Mitigation |
|---|---|
| **Tests execute real malicious commands** | Only run in isolated lab VM — never on production systems |
| **Some tests are destructive** | Always take a VMware snapshot before running tests |
| **Some tests download files from Internet** | Ensure VM has internet access for tests that require it |
| **Windows Defender may block tests** | Disable Defender for emulation exercises |
| **Some tests require cleanup** | Always run cleanup commands after each test |
| **Some tests require prerequisites** | Check prerequisites before running — missing prereqs cause failures |

Before EVERY emulation session:

`VM menu > Snapshot > Take Snapshot > "Pre-Exercise Clean State"`

After exercise is complete, restore to snapshot before next exercise.

This ensures every exercise starts from a known clean baseline.

---

## Prerequisites

### Step 1 — Verify Lab Environment is Ready

On **WIN11-CLIENT01**, open PowerShell as Administrator and verify below:

```powershell
# Verify Sysmon is running
Get-Service Sysmon64 | Select-Object Status, Name

# Verify Wazuh Agent is running and connected
Get-Service WazuhSvc | Select-Object Status, Name

# Verify PowerShell execution policy
Get-ExecutionPolicy -Scope LocalMachine

# Verify C:\Tools directory exists
Test-Path C:\Tools
```

![Image](/images/lab-setup/atomic-red-team/01-prerequisite.png)

---

## Part 1 — Install Invoke-AtomicRedTeam

### Step 1 — Atomic Red Team Installation with Explicit Paths

```powershell
# Add a Defender exclusion for C:\AtomicRedTeam
Add-MpPreference -ExclusionPath "C:\AtomicRedTeam"

# Verify the added exclusion
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath

# Load Atomic Red Team installer function
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)

# Check if the installer function was loaded
Get-Command Install-AtomicRedTeam

# Install Atomic Red Team
Install-AtomicRedTeam -InstallPath "C:\AtomicRedTeam" -getAtomics -Force
```

![Image](/images/lab-setup/atomic-red-team/02-installation.png)

---

## Part 2 — Configure Invoke-AtomicRedTeam

### Step 1 — Import the Module

```powershell
# Import Invoke-AtomicRedTeam module
Import-Module C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1 -Force
```

### Step 2 — Add Module to PowerShell Profile

Adding to the profile means the module loads automatically every time
PowerShell opens — we will not need to import manually for each session:

```powershell
# Check if profile exists:
Test-Path $PROFILE

# If False — create it:
New-Item -ItemType File -Path $PROFILE -Force

# Add auto-import:
"Import-Module C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1 -Force" | Out-File -Append $PROFILE

# Verify profile content:
Get-Content $PROFILE
```

![Image](/images/lab-setup/atomic-red-team/03-installation.png)

### Step 3 — Verify Installation

```powershell
# Verify framework installed
Test-Path C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1

# Verify atomics library installed
Test-Path C:\AtomicRedTeam\atomics

# Count available techniques
(Get-ChildItem C:\AtomicRedTeam\atomics -Directory).Count
```

![Image](/images/lab-setup/atomic-red-team/04-installation.png)

---

## Part 3 — Understanding Atomic Tests

### Step 1 — Explore the Atomics Directory Structure

View atomics directory structure:

```powershell
Get-ChildItem C:\AtomicRedTeam\atomics | Select-Object Name -First 20
```

![Image](/images/lab-setup/atomic-red-team/05-art-test.png)

> Each directory corresponds to one ATT&CK technique.

### Step 2 — View Test Details Before Executing

**Always read test details before running any atomic test.**

List all details for a technique:

```powershell
Invoke-AtomicTest T1059.001 -ShowDetails
```

![Image](/images/lab-setup/atomic-red-team/06-art-test.png)

### Step 3 — List All Tests for a Technique

Some techniques have multiple tests. List them all with their numbers:

```powershell
Invoke-AtomicTest T1059.001 -ShowDetailsBrief
```

![Image](/images/lab-setup/atomic-red-team/07-art-test.png)

---

## Part 4 — Take Snapshot

### Step 1 — Take Post-Installation Snapshot

Before running any exercises, take a clean snapshot with ART installed:

Lock screen: `Win + L`

In VMware: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `ART Installed - Ready for Exercises` |
| Description | `Sysmon running. Wazuh Agent connected. Atomic Red Team installed at C:\AtomicRedTeam. Ready for Purple Team exercises.` |

![Image](/images/lab-setup/atomic-red-team/08-snapshot.png)

---

### Part 5 — Disable Defender for Emulation Testing

### Step 1 — Check Defender Status

```powershell
Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled, BehaviorMonitorEnabled, IoavProtectionEnabled, IsTamperProtected
```

![Image](/images/lab-setup/atomic-red-team/09-art-test.png)

### Step 2 — Turn off Tamper Protection

On **WIN11-CLIENT01**:

- Open `Windows Security`
- Click on `Virus & threat protection`
- Click on `Manage settings` under `Virus & threat protection settings`
- Scroll to `Tamper Protection`
- Turn it Off
- Accept the UAC prompt

> This setting can only be changed from the Windows Security UI
> (or through enterprise management such as Microsoft Intune).
> PowerShell cannot disable Tamper Protection. Microsoft intentionally
> designed it this way to prevent malware (or administrators) from
> disabling Defender programmatically.

### Step 3 — Disable Defender Components

After Tamper Protection is off, open PowerShell as Administrator and run:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableBehaviorMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
```

> `RealtimeMonitoring` & `BehaviorMonitoring` settings are not persistent across reboots.
> Microsoft Defender is designed to automatically re-enable protection after a restart,
> even if Tamper Protection is disabled. The Set-MpPreference commands change the
> configuration for the current session, but they are not intended to permanently
> disable Defender.

Verify Defender status:
```powershell
Get-MpComputerStatus | Select-Object RealTimeProtectionEnabled, BehaviorMonitorEnabled, IoavProtectionEnabled, IsTamperProtected
```

![Image](/images/lab-setup/atomic-red-team/10-art-test.png)

Disable automatic sample submission:
```powershell
# Check current status
Get-MpPreference | Select-Object SubmitSamplesConsent

# Disable automatic sample submission
Set-MpPreference -SubmitSamplesConsent 2
```

![Image](/images/lab-setup/atomic-red-team/11-art-test.png)

The **`SubmitSamplesConsent`** setting controls how Microsoft Defender handles suspicious file submissions to Microsoft for further analysis. The following values are supported:

| Value | Meaning | Description |
|---|---|---|
| **0** | Always Prompt | Prompts the user before submitting any suspicious file to Microsoft. |
| **1** | Send Safe Samples Automatically | Automatically submits files that Microsoft considers safe to upload, while prompting for other files. |
| **2** | Never Send | Prevents Microsoft Defender from automatically submitting suspicious files. |
| **3** | Send All Samples Automatically | Automatically submits all suspicious files to Microsoft without prompting the user. |

> **Note:** For isolated malware analysis or detection engineering labs,
> setting **`SubmitSamplesConsent`** to **`Never Send (2)`** prevents
> Microsoft Defender from automatically uploading test files or attack
> simulation artifacts to Microsoft. This helps ensure Atomic Red Team
> tests and other lab activities remain within the local environment.

---

## Part 6 — Running Atomic Tests

### Step 1 — Check Prerequisites Before Running

Check if prerequisites are met before running:

```powershell
Invoke-AtomicTest T1053.005 -TestNumbers 1 -CheckPrereqs
```

![Image](/images/lab-setup/atomic-red-team/12-art-test.png)

If prerequisites are not met, install them using:

```powershell
Invoke-AtomicTest T1053.005 -TestNumbers 1 -GetPrereqs
```

### Step 2 — Run First Atomic Test

When a technique has multiple tests, we can specify which one to run.

```powershell
# Run all tests within a technique
Invoke-AtomicTest T1053.005

# Run test number 1 only
Invoke-AtomicTest T1053.005 -TestNumbers 1

# Run test numbers 1 and 3
Invoke-AtomicTest T1053.005 -TestNumbers 1,3

# Run test with verbose output
Invoke-AtomicTest T1053.005 -TestNumbers 1 -ShowDetails
```

For testing purpose, we will run TestNumbers `1` under technique `T1053.005`.

![Image](/images/lab-setup/atomic-red-team/13-art-test.png)

We can manually verify that the tasks were created:

```powershell
Get-ScheduledTask -TaskName T1053_005*
```

![Image](/images/lab-setup/atomic-red-team/14-art-test.png)

---

## Part 7 — Verify Alert on Wazuh Dashboard

Open browser on Windows 11 host: `https://192.168.48.129`

Navigate to: `Left sidebar > Threat intelligence > Threat Hunting > Events`

### Step 1 — Filter for Sysmon Alerts

On Wazuh Dashboard, filter for sysmon alerts: `rule.groups: sysmon`

![Image](/images/lab-setup/atomic-red-team/15-art-test.png)

### Step 2 — Verify Alert Details

Click on the magnifying lens to see the alert details:

![Image](/images/lab-setup/atomic-red-team/16-art-test.png)

We can see that tasks "**T1053_005_OnLogon**" and "**T1053_005_OnStartup**" were created as mentioned in test execution output.

---

## Part 8 — Cleanup After Tests

### Step 1 — Why Cleanup Matters

Some atomic tests make persistent changes to the system:

| Test | What It Changes |
|---|---|
| T1547.001 | Adds registry Run key |
| T1053.005 | Creates scheduled task |
| T1003.001 | May drop files to disk |

Without cleanup, these changes accumulate and can affect subsequent tests.

### Step 2 — Run Cleanup

```powershell
# Clean up after a specific test
Invoke-AtomicTest T1053.005 -TestNumbers 1 -Cleanup

# Clean up all tests for a technique
Invoke-AtomicTest T1053.005 -Cleanup
```

![Image](/images/lab-setup/atomic-red-team/17-art-test.png)

> **Alternative:** Restore VMware snapshot instead of running cleanup.
> Snapshot restore guarantees a completely clean state.

---
