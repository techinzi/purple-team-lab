# Exercise 01 — Process Discovery

## Objective

Execute a realistic Process Discovery technique using Atomic Red Team,
capture endpoint telemetry via Sysmon and Windows Event Logs, validate
detection coverage in Wazuh, identify detection gaps, and document
findings.

---

## MITRE ATT&CK Mapping

| Category | Value |
|---|---|
| Tactic | Discovery |
| Technique | Process Discovery |
| Technique ID | T1057 |
| Sub-technique | None |
| Atomic Test | T1057-2 Process Discovery - tasklist |

---

## Prerequisites

Verify the following on **WIN11-CLIENT01** before starting the exercise:

```powershell
# Verify Sysmon is running
Get-Service Sysmon64

# Verify Wazuh Agent is connected
Get-Service WazuhSvc

# Verify Atomic Red Team module is loaded
Get-Command Invoke-AtomicTest
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/01-prerequisite.png)

Take a VMware snapshot before proceeding: `VMware > VM menu > Snapshot > Take Snapshot > Pre-Exercise`

---

## Part 1 — Attack Simulation

### Step 1 — Review Test Details

```powershell
# Check all available tests under T1057
Invoke-AtomicTest T1057 -ShowDetailsBrief

# Verify test details before execution
Invoke-AtomicTest T1057 -ShowDetails -TestNumbers 2
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/02-prerequisite.png)

> Review the attack description and command.

**What this executes under the hood:**

```cmd
tasklist
```

`tasklist.exe` is a native Windows binary that enumerates all running
processes and their associated metadata including PID, session name,
memory usage, and status.

**Attacker objective:**

An attacker uses Process Discovery immediately after initial access or
privilege escalation to:

- Identify running security tools (AV, EDR, monitoring agents)
- Locate target processes for injection (e.g. `lsass.exe`, `explorer.exe`)
- Confirm presence of specific applications containing sensitive data
- Determine if the system is being monitored or analysed

### Step 2 — Check Prerequisites

```powershell
Invoke-AtomicTest T1057 -CheckPrereqs -TestNumbers 2
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/03-prerequisite.png)

> Prerequisite is met.

### Step 3 — Execute the Attack

Open PowerShell as **Administrator** on **WIN11-CLIENT01** and execute **Test#2**:

```powershell
Get-Date
Invoke-AtomicTest T1057 -TestNumbers 2
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/04-attack.png)

---

## Part 2 — Telemetry Collection

### Step 1 — Confirm Sysmon Captured the Activity

**Relevant Event IDs:**

| Event ID | Name | Relevance |
|---|---|---|
| EID 1 | Process Creation | Primary detection source — captures `tasklist.exe` execution with full command line, parent process, and hashes |


**PowerShell — Collect Sysmon Evidence:**

```powershell
# Set start time as per test execution
$testStartTime = Get-Date "2026-07-14 19:00:00"

# Search for generated Sysmon events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Where-Object { $_.Id -eq 1 -and $_.Message -like "*tasklist*" -and $_.TimeCreated -gt $testStartTime } | Select-Object TimeCreated, Id, Message | Format-List
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/05-sysmon.png)

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/06-sysmon.png)

> For a single command execution, we found two logs becuase every process
> creation generates an individual log:
>
> Command: `"cmd.exe" /c tasklist`
>
> | Process | Parent Process |
> |---|---|
> | cmd.exe | powershell.exe |
> | tasklist.exe | cmd.exe |

---

### Step 2 — Confirm Windows Event Logs Captured the Activity

**Relevant Log:**

| Log | Event ID | Description |
|---|---|---|
| Security | 4688 | Process creation |

**PowerShell — Collect Windows Security Log Evidence:**

```powershell
# Set start time as per test execution
$testStartTime = Get-Date "2026-07-14 19:00:00"

# Check for EID 4688 process creation
Get-WinEvent -LogName Security | Where-Object { $_.Id -eq 4688 -and $_.TimeCreated -gt $testStartTime -and $_.Message -like "*tasklist*" } | Select-Object TimeCreated, Id, Message | Format-List
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/07-windowsLog.png)

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/08-windowsLog.png)

---

### Step 3 — Confirm Wazuh Detected the Activity

On Wazuh Dashboard, go to `Left sidebar > Threat intelligence > Threat Hunting > Events`.

Set time range as per test execution.

Search for **tasklist** execution: `data.win.eventdata.image: *tasklist*`

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/09-wazuh.png)

One Wazuh rule detected the activity. Open the alert details:

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/10-wazuh.png)

This exercise triggered **Wazuh Rule 92032 – Suspicious Windows cmd shell execution**.

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/11-wazuh.png)

The rule monitors **Sysmon Event ID 1 (Process Creation)** and looks for processes launched as:

```powershell
cmd.exe /c <command>
```

The detection is based on two conditions:

- Parent process is `cmd.exe`
- Parent command line contains `/c`

This rule is intentionally low severity (Level=3) because `cmd.exe /c` is commonly used by both legit users and attackers. It serves as a behavioural indicator that should be correlated with additional telemetry such as the executed command, parent process, user context, and subsequent activity.

---

## Part 3 — Detection Engineering

### Existing Wazuh Detection

This exercise triggered **Wazuh Rule 92032 – Suspicious Windows cmd shell execution**.

---

### Detection Gaps

No

---

### Detection Improvements
Not required.

---

### Sigma Rule

Not required.

---

## Part 4 — MITRE ATT&CK Coverage

### Update ATT&CK Navigator Layer

1. Open ATT&CK Navigator at `http://192.168.48.129:4200`
2. Create a new layer for the first time.
3. Find **T1057 — Process Discovery** in the **Discovery** tactic column
4. Set color: Green (telemetry present and alerted)
6. Add comment: `Tested using "AtomicTest T1057 - Test#2"`
7. Export updated layer as JSON
8. Save as `coverage-validation-status.json`

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/12-navigator.png)

---

## Part 5 — Cleanup

### Option 1: Restore VMware Snapshot (Preferred)

`VMware > VM menu > Snapshot > Snapshot Manager > Select "Pre-Exercise" > Click Restore`

This guarantees a completely clean baseline for the next exercise.

### Option 2: Manual Cleanup

```powershell
Invoke-AtomicTest T1057 -TestNumbers 2 -Cleanup
```

![Image](/images/atomic-red-team-exercises/01-process-discovery-t1057/13-cleanup.png)

> **Note:** T1057 Test #2 (`tasklist`) is non-persistent and leaves
> no artifacts on disk or in the registry. The Atomic Red Team cleanup
> command handles any output files created during execution.

---
