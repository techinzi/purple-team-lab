# MITRE Caldera — Installation & Configuration Guide

## Overview

**MITRE Caldera** is an open-source adversary emulation platform that
automates multi-step attack operations mapped to the MITRE ATT&CK
framework. Unlike **Atomic Red Team** which executes single isolated
techniques, Caldera orchestrates complete attack chains — deploying
agents to endpoints and running sequences of techniques that simulate
real adversary behavior.

### Caldera vs Atomic Red Team — When to Use Each

| Scenario | Use Atomic Red Team | Use Caldera |
|---|---|---|
| Test one specific technique | ✅ | — |
| Validate a single detection rule | ✅ | — |
| Quick coverage testing | ✅ | — |
| Simulate a multi-step attack chain | — | ✅ |
| Emulate a named threat actor | — | ✅ |
| Automate sequential technique execution | — | ✅ |
| Realistic adversary simulation | — | ✅ |

### Core Caldera Concepts

| Concept | Definition |
|---|---|
| **Agent** | Lightweight implant deployed to the target endpoint (WIN11-CLIENT01) that receives and executes commands from the Caldera server |
| **Ability** | A single ATT&CK-mapped action that an agent can perform (equivalent to one atomic test) |
| **Adversary** | A collection of abilities grouped to simulate a threat actor's behavior |
| **Operation** | A running execution of an adversary profile against a group of agents |
| **Fact** | Information discovered during an operation (e.g. username, IP address) used by subsequent abilities |
| **Planner** | The decision engine that determines which abilities to run and in what order |

---

## Hardware and Software Requirements

| Resource | Required | Notes |
|---|---|---|
| RAM | 2 GB additional | On top of existing Wazuh usage |
| Disk | ~2 GB | Caldera plus plugins |
| Python | 3.8 or higher | Ubuntu 22.04 ships with 3.10 |
| pip | Latest | For Python package installation |
| Port | 8888 (HTTP) | Caldera web UI |
| Port | 8443 (HTTPS) | Caldera secure UI (optional) |
| Port | 7010 (TCP) | Agent beacon communication |
| Port | 7011 (UDP) | Agent beacon communication |
| Port | 7012 (TCP) | Agent file transfer |

> **Memory Warning:** Running Caldera alongside Wazuh on 4 GB RAM
> will be tight. Before starting Caldera, verify at least 1 GB is
> available:
>
> ```bash
> # Run this on ubuntu-server01 via SSH
> free -h
>
> # Available column should show > 1 GB
> ```
>
> If available memory is less than 1 GB, reduce the Wazuh Indexer
> heap size first:
>
> ```bash
> sudo nano /etc/wazuh-indexer/jvm.options
>
> # Change -Xms and -Xmx values from 2g to 512m
>
> sudo systemctl restart wazuh-indexer
>
> # Wait 30 seconds then check memory again
> free -h
> ```

![Image](/images/lab-setup/caldera/01-prerequisite.png)

---

## Prerequisites

### Step 1 — Verify Python Version

```bash
python3 --version
pip3 --version
```

![Image](/images/lab-setup/caldera/02-prerequisite.png)

### Step 2 — Open Required Ports in UFW

Check which Caldera ports are already open:

```bash
sudo ufw status | grep -E '8888|7010|7011|7012'
```

![Image](/images/lab-setup/caldera/03-prerequisite.png)

> Port 8888 found to be already opened as it was configured during **ubuntu-server01** initial setup.

Verify nothing is actually running on above opened port:

```bash
sudo ss -tlnp | grep 8888
```

![Image](/images/lab-setup/caldera/04-prerequisite.png)

> If output is empty, nothing is listening on 8888 — port is open in UFW but unused.
> Caldera will use it. No conflict.

Allow ports that are missing:

```bash
sudo ufw allow 7010/tcp
sudo ufw allow 7011/udp
sudo ufw allow 7012/tcp
sudo ufw reload

# Verify all Caldera ports open
sudo ufw status | grep -E '8888|7010|7011|7012'
```

![Image](/images/lab-setup/caldera/05-prerequisite.png)

---

## Part 1 — Install Caldera 4.1.0

### Step 1 — Install System Dependencies

```bash
sudo apt-get update
sudo apt-get install -y python3-pip python3-dev build-essential git curl golang-go
```

![Image](/images/lab-setup/caldera/06-installation.png)

Verify Go installation (required for Sandcat agent compilation):

```bash
which go
go version
```

![Image](/images/lab-setup/caldera/07-installation.png)

> **Note:** Caldera 4.1.0 requires Go 1.17 minimum.

### Step 2 — Clone Caldera Repository

Install Caldera under `/opt` directory:

```bash
# Create directory
sudo mkdir -p /opt/caldera
sudo chown labadmin:labadmin /opt/caldera

# Clone Caldera with all plugins
git clone https://github.com/mitre/caldera.git --recursive --branch 4.1.0 /opt/caldera
```

![Image](/images/lab-setup/caldera/08-installation.png)

> **Why --recursive?**
>
> Caldera uses Git submodules for its plugins (Stockpile, Sandcat,
> Manx, etc.). Without `--recursive` the plugins directory is empty
> and Caldera will not function correctly.

Verify clone:

```bash
ls /opt/caldera
```

Verify plugins downloaded:

```bash
ls /opt/caldera/plugins/
```

![Image](/images/lab-setup/caldera/09-installation.png)

### Step 3 — Create a Python Virtual Environment

A Python virtual environment keeps CALDERA's dependencies isolated from the
Ubuntu system Python, preventing package conflicts and making upgrades easier.

```bash
cd /opt/caldera

# Install the Python virtual environment package (if not already installed)
sudo apt update
sudo apt install -y python3-venv
```

![Image](/images/lab-setup/caldera/10-installation-01.png)

```bash
# Create a virtual environment
python3 -m venv .venv

# Activate the virtual environment
source .venv/bin/activate
```

After activation, the shell prompt should look similar to: `(.venv) labadmin@ubuntu-server01:/opt/caldera$`

![Image](/images/lab-setup/caldera/10-installation-02.png)

### Step 4 — Install Python Dependencies

With the virtual environment activated, install the required Python packages:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

![Image](/images/lab-setup/caldera/11-installation-01.png)

> **Important:**
>
> Do **not** use `sudo pip3 install -r requirements.txt`.
>
> Installing packages with `sudo pip3` modifies the system Python environment and may
> introduce dependency conflicts with Ubuntu-managed packages. Installing dependencies
> inside the virtual environment keeps CALDERA isolated from the operating system.

Wait until all dependencies are installed successfully.

![Image](/images/lab-setup/caldera/11-installation-02.png)

Verify that Python is actually using the virtual environment:

```bash
which python
which pip
```

> The expected output should contain `.venv` in path.

![Image](/images/lab-setup/caldera/11-installation-03.png)

---

## Part 2 — Configure Caldera

### Step 1 — Review Default Configuration

```bash
cat /opt/caldera/conf/default.yml
```

![Image](/images/lab-setup/caldera/12-installation.png)

![Image](/images/lab-setup/caldera/13-installation.png)

Key settings to note:
- app.contact.http
- exfil_dir
- host
- port
- users

### Step 2 — Create Custom Configuration

Create a lab-specific configuration with stronger passwords:

```bash
cp /opt/caldera/conf/default.yml /opt/caldera/conf/local.yml
nano /opt/caldera/conf/local.yml
```

![Image](/images/lab-setup/caldera/14-installation.png)

Update the following settings:

```yaml
app.contact.http: http://192.168.48.129:8888

host: 0.0.0.0
port: 8888

users:
  blue:
    blue: 'ChangeThisBluePassword'		# Change this
  red:
    admin: 'ChangeThisRedAdminPassword'	# Change this
    red: 'ChangeThisRedPassword'		# Change this

plugins:
- access
- atomic
- compass
- debrief
- fieldmanual
- manx
- response
- sandcat
- stockpile
- training
```

Save: `Ctrl + O > Enter > Ctrl + X`

> **Why app.contact.http matters:**
>
> Caldera uses this address in all generated agent deployment commands.
> Replace `0.0.0.0` with the actual IP address of **ubuntu-server01**
> so that deployed agents can connect successfully.

---

## Part 3 — Start Caldera

### Step 1 — Start Caldera Manually First

Test that Caldera starts correctly before configuring as a service:

```bash
# Navigate to the Caldera directory (if not already there)
cd /opt/caldera

# Activate virtual environment (if not already)
source .venv/bin/activate

# See Caldera help menu
python3 server.py --help

# Start Caldera in virtual environment
python3 server.py -E local
```

![Image](/images/lab-setup/caldera/15-installation.png)

![Image](/images/lab-setup/caldera/16-installation.png)

Once we see `All systems ready`, the server is idle and waiting for connections.

> **Keep this terminal open.** Open a second SSH session for
> subsequent commands. Caldera runs in the foreground.

### Step 2 — Access Caldera Web Interface

Open browser on Windows 11 host:

```
http://192.168.48.129:8888
```

Login page should have appeared but we have encountered below error:

![Image](/images/lab-setup/caldera/17-installation.png)

### Step 3 — Troubleshooting CALDERA 4.1.0: 500 Internal Server Error on Ubuntu 22.04

#### Problem

After successfully starting CALDERA using `python3 server.py -E local`, the server
appeared to start normally and displayed `All systems ready.`

However, browsing to the web interface returned:

```text
500 Internal Server Error
Server got itself in trouble
```

The browser never displayed the login page.

#### Root Cause

CALDERA 4.1.0 depends on `aiohttp 3.8.1`. 

During installation, pip resolved some of aiohttp's dependencies to newer versions:

- `yarl`
- `multidict`
- `aiosignal`

These package versions are not fully compatible with **CALDERA 4.1.0**, which was released in 2022.

Although `aiohttp` itself remained at the correct version (`3.8.1`), these newer
dependency versions introduced compatibility changes that caused CALDERA 4.1.0
to return **500 Internal Server Error** when accessing the web interface.

Only the dependency packages required downgrading; `aiohttp==3.8.1` itself did
not need to be changed.

Specifically, newer versions of `yarl` introduce stricter URL validation that causes CALDERA 4.1.0 to fail while processing incoming HTTP requests.

#### Verify Installed Versions

```bash
pip freeze | grep -E "aiohttp|yarl|multidict|aiosignal"
```

Highlighted are the problematic versions:

![Image](/images/lab-setup/caldera/18-troubleshooting.png)

#### Resolution

```bash
# Remove the incompatible packages
cd /opt/caldera
pip uninstall -y yarl multidict aiosignal

# Install versions compatible with CALDERA 4.1.0
pip install yarl==1.7.2 multidict==5.2.0 aiosignal==1.2.0

# Verify the installed versions
pip freeze | grep -E "aiohttp|yarl|multidict|aiosignal"
```

![Image](/images/lab-setup/caldera/19-troubleshooting.png)

#### Restart CALDERA

Stop the running server: `Ctrl + C`

Start CALDERA again: `python3 server.py -E local`

![Image](/images/lab-setup/caldera/20-troubleshooting.png)

When the server displays `All systems ready`, browse to `http://192.168.48.129:8888/`

The login page should now load successfully:

![Image](/images/lab-setup/caldera/21-troubleshooting.png)

Login with red team credentials:

| Field | Value |
|---|---|
| Username | `red` |
| Password | `ChangeThisRedPassword` |

Click **Login**.

The Caldera home page loads showing:

![Image](/images/lab-setup/caldera/22-troubleshooting.png)

#### Why this Fix Works

CALDERA 4.1.0 was developed and tested against dependency versions available in 2022.

Modern versions of `yarl` (1.20+) introduce stricter host validation that is incompatible with CALDERA 4.1.0's request handling.

Downgrading the following packages restores compatibility:

| Package | Compatible Version |
|---|---|
| yarl | 1.7.2 |
| multidict | 5.2.0 |
| aiosignal | 1.2.0 |

No changes to the CALDERA source code are required.

### Step 4 — Stop Manual Instance

Return to the terminal running Caldera and press **Ctrl + C** to stop it.

---

## Part 4 — Run Caldera as Background Service

### Step 1 — Create Caldera systemd Service

Running Caldera as a systemd service means it starts automatically
on boot and runs independently of any terminal session.

Exit out of the virtual environment (if not already):

```bash
deactivate
```

Create the service file **caldera.service** using below command:

```bash
sudo tee /etc/systemd/system/caldera.service > /dev/null << 'EOF'
[Unit]
Description=MITRE CALDERA Adversary Emulation Platform
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=labadmin
Group=labadmin

WorkingDirectory=/opt/caldera

# Use the Python interpreter from the virtual environment
ExecStart=/opt/caldera/.venv/bin/python /opt/caldera/server.py -E local

Restart=on-failure
RestartSec=10

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF
```

![Image](/images/lab-setup/caldera/23-creating-service.png)

### Step 2 — Enable and Start Caldera Service

```bash
# Reload systemd to recognize new service
sudo systemctl daemon-reload

# Enable Caldera to start on boot
sudo systemctl enable caldera

# Start Caldera service
sudo systemctl start caldera

# Wait for startup
sleep 15

# Check status
sudo systemctl status caldera
```

![Image](/images/lab-setup/caldera/24-creating-service.png)

### Step 3 — Verify Caldera is Listening

```bash
sudo ss -tlnp | grep 8888
```

![Image](/images/lab-setup/caldera/25-creating-service.png)

Open a browser and navigate to `http://192.168.48.129:8888/`.

The CALDERA login page should load successfully.

### Step 4 — Check Caldera Service Logs

```bash
sudo journalctl -u caldera -n 20 --no-pager
```

Look for: `All systems ready`

![Image](/images/lab-setup/caldera/26-creating-service.png)

### Step 5 — Validate Caldera Service Survives Reboot

Reboot **ubuntu-server01**:

```bash
sudo reboot
```

![Image](/images/lab-setup/caldera/27-creating-service.png)

After the server comes back online, reconnect via SSH.

```bash
# Verify the service status
sudo systemctl is-active caldera

# Verify CALDERA is listening on port 8888
sudo ss -tlnp | grep 8888
```

![Image](/images/lab-setup/caldera/28-creating-service.png)

Finally, open a web browser and navigate to `http://192.168.48.129:8888/`.

The CALDERA login page should load successfully without requiring any manual intervention.

---

## Part 5 — Explore the Caldera Interface

### Step 1 — Navigate Core Sections

Login to Caldera interface at `http://192.168.48.129:8888`.

**Agents:** `Left sidebar` > `CAMPAIGNS` > `agents`

Currently empty — agents will be deployed in the next section.

![Image](/images/lab-setup/caldera/29-explore-caldera.png)

**Abilities:** `Left sidebar` > `CAMPAIGNS` > `abilities`

Shows all available ATT&CK-mapped actions. Filter by tactic to explore.

![Image](/images/lab-setup/caldera/30-explore-caldera.png)

**Adversaries:** `Left sidebar` > `CAMPAIGNS` > `adversaries`

Shows built-in adversary profiles.

![Image](/images/lab-setup/caldera/31-explore-caldera.png)

**Operations:** `Left sidebar` > `CAMPAIGNS` > `operations`

It is where we create and monitor running attack chains.

![Image](/images/lab-setup/caldera/32-explore-caldera.png)

---

## Part 6 — Take Snapshot

### Step 1 — Take Snapshot After Caldera Installation

```bash
# On ubuntu-server01, clean up before snapshot
history -c && history -w
sudo apt-get clean

# Close SSH session
exit
```

![Image](/images/lab-setup/caldera/33-snapshot.png)

In VMware: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `Caldera Installed` |
| Description | `Wazuh running. ATT&CK Navigator running. Caldera 4.1.0 installed.` |

![Image](/images/lab-setup/caldera/34-snapshot.png)

---

## Part 7 — Deploy Sandcat Agent to Windows VM

The Sandcat agent is Caldera's default implant. It runs on the target
endpoint (WIN11-CLIENT01) and receives commands from the Caldera server.

### Step 1 — Generate Agent Deployment Command

In the Caldera web interface, navigate to:

`Left sidebar` > `CAMPAIGNS` > `agents` > `Deploy an agent`

![Image](/images/lab-setup/caldera/35-deploy-agent.png)

Select agent as `Sandcat` and platform as `Windows`

> **Note:** The agent implant is named `splunkd` by default —
> this is intentional. Caldera names agents after legitimate-sounding
> processes to simulate real adversary masquerading behavior.
> Leaving it as is for lab purposes.

Caldera generates a variety of PowerShell deployment command:

![Image](/images/lab-setup/caldera/36-deploy-agent.png)

Copy `CALDERA's default agent` PowerShell command exactly from the Caldera UI.

### Step 2 — Deploy Agent on WIN11-CLIENT01

On WIN11-CLIENT01, open PowerShell as Administrator.

Disable Defender real-time monitoring:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

Execute the agent deployment copied command and wait for execution to complete.

![Image](/images/lab-setup/caldera/37-deploy-agent.png)

When prompmted for allowing the implant, **allow** it:

![Image](/images/lab-setup/caldera/38-deploy-agent.png)

### Step 3 — Verify Agent Appears in Caldera

Return to Caldera web interface: `Left sidebar` > `CAMPAIGNS` > `agents`

WIN11-CLIENT01 should appear in few seconds:

![Image](/images/lab-setup/caldera/39-deploy-agent.png)

Click on the WIN11-CLIENT01 agent to see its details:

![Image](/images/lab-setup/caldera/40-deploy-agent.png)

---

## Part 8 — Run an Initial Operation

### Step 1 — Create a New Operation

> **Note:** Ensure the Sandcat agent is still running and shown as `alive`
> in the `agents` page before creating an operation. Operations require at
> least one active agent to execute abilities.

In the Caldera web interface, navigate to:

`Left sidebar` > `CAMPAIGNS` > `operations` > `Create Operation`

![Image](/images/lab-setup/caldera/41-run-operation.png)

Provide basic required details:

| Field | Value |
|---|---|
| Name | `Discovery Exercise 01` |
| Adversary | `Discovery` |
| Fact Source | `basic` |
| Group | `red` |
| Planner | `atomic` |

Click **Start**.

![Image](/images/lab-setup/caldera/42-run-operation.png)

### Step 2 — Monitor Operation in Real Time

The operation dashboard shows:

![Image](/images/lab-setup/caldera/43-run-operation.png)

For each ability, we can click on `View Command` to see the executed command and `View Output` to see the command output.


### Step 3 — Wait for Operation to Complete

Consider the operation as completed once every scheduled ability has finished with
either a `Success` or `Failed` status and no abilities are still Running.
In CALDERA 4.1.0, the operation may continue to display a `running` status
even after execution has completed.

![Image](/images/lab-setup/caldera/44-run-operation.png)

> Note that as per the Caldera Operation timeline, activities were
> performed between 11:09 AM to 11:15 AM.

---

## Part 9 — Verify Operation Results

### Step 1 — Review Operation Results in Caldera

Navigate to: `Left sidebar` > `CAMPAIGNS` > `operations` > `Select an operation`

![Image](/images/lab-setup/caldera/45-run-operation.png)

Click on `Download` to download the report:

![Image](/images/lab-setup/caldera/46-run-operation.png)

Available export formats:
- Full Report (JSON)
- Event Logs (JSON)
- CSV

![Image](/images/lab-setup/caldera/47-run-operation.png)

In CALDERA v4.1.0, the `Full Report` export may contain only `null` even after
a successful operation.

Use the `Event Logs` or `CSV` export instead, as both contain the complete operation execution details and can be used for analysis.

![Image](/images/lab-setup/caldera/48-run-operation.png)

> NOTE: First command executed was `$env:username`

### Step 2 — Verify Sysmon Captured Operation Activity

As per the Caldera Operation timeline, activities were performed between
11:09 AM to 11:15 AM.

On **WIN11-CLIENT01**, run below in PowerShell to export sysmon logs (Event ID 1 - Process Creation) generated during Caldera operation:

```powershell
$operationStart = Get-Date "2026-07-05 11:05:00"
$operationEnd   = Get-Date "2026-07-05 11:20:00"

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" |
Where-Object {
    $_.TimeCreated -ge $operationStart -and
    $_.TimeCreated -le $operationEnd -and
    $_.Id -eq 1
} |
ForEach-Object {
    $xml = [xml]$_.ToXml()

    [PSCustomObject]@{
        Time        = $_.TimeCreated
        Image       = ($xml.Event.EventData.Data | Where-Object Name -eq 'Image').'#text'
        CommandLine = ($xml.Event.EventData.Data | Where-Object Name -eq 'CommandLine').'#text'
    }
} | Export-Csv "$env:USERPROFILE\Desktop\sysmon-processes.csv" -NoTypeInformation
```

![Image](/images/lab-setup/caldera/49-run-operation.png)

CSV shows commands from discovery techniques:

![Image](/images/lab-setup/caldera/50-run-operation.png)

> NOTE: We see the same `$env:username` command in Caldera report and sysmon log export.

### Step 3 — Verify Operation Alerts in Wazuh Dashboard

Open Wazuh on Windows 11 host: `https://192.168.48.129`

Navigate to: `Left sidebar > Threat intelligence > Threat Hunting > Events`

Apply filter:
- agent.name: `WIN11-CLIENT01`
- rule.groups: `sysmon`

As per the Caldera Operation timeline, activities were performed between
11:09 AM to 11:15 AM. Set time range in Wazuh accordingly and search.
We should see multiple Sysmon alerts executed during the Caldera operation:

![Image](/images/lab-setup/caldera/51-run-operation.png)

Click on the `Discovery` sysmon alert to see its details:

![Image](/images/lab-setup/caldera/52-run-operation.png)

> Above Wazuh alert shows the Sandcat agent (`splunkd.exe`) connected to
> the CALDERA server at `192.168.48.129:8888` as a member of the `red` group.
> CALDERA then instructed the agent to launch PowerShell, which executed
> the WMI command to enumerate installed antivirus products.

---

## Part 10 — Clean Up the Sandcat Agent

After completing all verification steps, remove the deployed Sandcat agent
to leave the lab in a clean state.

On **WIN11-CLIENT01**, terminate the agent if it is still running:

```powershell
# Check if agent running
Get-Process splunkd

# Terminate the agent
Stop-Process -Name splunkd -Force -ErrorAction SilentlyContinue
```

![Image](/images/lab-setup/caldera/53-cleanup.png)

Delete the agent executable:

```powershell
# Verify file exist
Test-Path "C:\Users\Public\splunkd.exe"

# Delete the file
Remove-Item "C:\Users\Public\splunkd.exe" -Force -ErrorAction SilentlyContinue
```

![Image](/images/lab-setup/caldera/54-cleanup.png)

In the CALDERA web interface:

1. Navigate to **Agents**.
2. Select the deployed agent.
3. Delete the agent entry.

> **Recommended:** Restore the `WIN11-CLIENT01` VMware snapshot after completing
> the exercise. This returns the endpoint to a known-clean state and removes all
> artifacts generated during the operation, including the Sandcat agent,
> temporary files, and any changes made during execution.

---
