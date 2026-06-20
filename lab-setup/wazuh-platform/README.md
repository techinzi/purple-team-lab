# Wazuh Platform — Installation & Configuration Guide

## Overview

Wazuh is an open-source security platform that provides Security Information
and Event Management (SIEM), endpoint detection, and threat response
capabilities. In the Purple Team home lab, Wazuh serves as the central
telemetry collection and detection validation platform.

### Role in Purple Team Lab

| Function | Description |
|---|---|
| **Log Collection** | Receives logs from all VMs via Wazuh agents |
| **Telemetry Analysis** | Processes Sysmon events, Windows Event Logs, and Linux logs |
| **Detection Validation** | Confirms whether alerts fire after adversary emulation exercises |
| **Coverage Measurement** | Provides evidence for ATT&CK coverage analysis |
| **Alert Management** | Generates and manages security alerts for Purple Team review |

### Purple Team Workflow

```
Windows 11 VM (WIN11-CLIENT01)
├── Sysmon generates telemetry
├── Wazuh Agent forwards logs
└── ──────────────────────────────────────────►
                                               Wazuh Platform (ubuntu-server01)
                                               ├── Wazuh Manager	— processes logs
                                               ├── Wazuh Indexer	— stores events
                                               └── Wazuh Dashboard	— visualizes alerts
                                                         │
                                                         ▼
                                               Purple Team Analyst
                                               validates detections
```

---

## Architecture

Wazuh consists of three core components, all installed on **ubuntu-server01**:

| Component | Role | Port |
|---|---|---|
| **Wazuh Manager** | Receives logs from agents, applies detection rules, generates alerts | 1514, 1515, 1516, 55000 |
| **Wazuh Indexer** | Stores and indexes all events and alerts (OpenSearch-based) | 9200, 9300 |
| **Wazuh Dashboard** | Web UI for viewing alerts, agents, and analytics | 443 |

Wazuh Port Reference:

| Port | Component | Purpose |
|---|---|---|
| `443` | Wazuh Dashboard | Web UI access |
| `1514` | Wazuh Manager | Agent log forwarding |
| `1515` | Wazuh Manager | Agent enrollment |
| `1516` | Wazuh Manager | Wazuh cluster daemon |
| `55000` | Wazuh Manager | REST API |
| `9200` | Wazuh Indexer | REST API and data ingestion |
| `9300` | Wazuh Indexer | Node-to-node transport |

```
                    ┌─────────────────────────────────────┐
                    │         ubuntu-server01             │
                    │                                     │
  Wazuh Agent  ────►│  Wazuh Manager  ──►  Wazuh Indexer  │
  (WIN11)           │       │                    │        │
                    │       └──────────────────► │        │
                    │                    Wazuh Dashboard  │
                    └─────────────────────────────────────┘
                                    │
                                    ▼
                            Browser (WIN11-CLIENT01)
                         https://192.168.48.129 (Port 443)
```

---

## Hardware Requirements

| Resource | Minimum | Recommended | This Lab |
|---|---|---|---|
| CPU | 2 cores | 4 cores | 2 cores |
| RAM | 4 GB | 8 GB | 4 GB |
| Disk | 50 GB | 100 GB | 80 GB |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS | Ubuntu 22.04.5 LTS |

> **Note on 4 GB RAM:**
> Wazuh officially recommends 8 GB RAM for production. For a home lab with
> limited log volume (1-2 agents), 4 GB is sufficient. Expect occasional
> slowness when all three components are indexing simultaneously during
> heavy emulation exercises.

---

## Prerequisites

### Network Requirements

| Item | Value |
|---|---|
| Ubuntu Server IP | 192.168.48.129 |
| Internet access | Required for package download |
| Ports open on UFW | 1514, 1515, 1516, 55000, 9200, 9300, 443 |

### Verify Prerequisites Before Starting

```bash
# Verify Ubuntu version
lsb_release -a

# Verify static IP is configured
ip addr show | grep inet

# Verify internet connectivity
ping -c 4 8.8.8.8

# Verify minimum disk space (50 GB free required)
df -h /

# Verify UFW ports are open
sudo ufw status
```

![Image](/images/lab-setup/wazuh-platform/01-pre-requisite.png)

![Image](/images/lab-setup/wazuh-platform/02-pre-requisite.png)

Confirm ports 1514, 1515, 1516, 55000, 9200, 9300 and 443 are allowed.
If not, add them:

```bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 1516/tcp
sudo ufw allow 55000/tcp
sudo ufw allow 9200/tcp
sudo ufw allow 9300/tcp
sudo ufw allow 443/tcp
sudo ufw reload
sudo ufw status
```

![Image](/images/lab-setup/wazuh-platform/03-1-pre-requisite.png)
![Image](/images/lab-setup/wazuh-platform/03-2-pre-requisite.png)

---

## Part 1 — Install Wazuh Using Official Assistant Script

Wazuh provides an official installation assistant script that installs and
configures all three components (Manager, Indexer & Dashboard) automatically.
This is the recommended installation method for home lab environments.

### Step 1 — Download Wazuh Installation Assistant & Configuration File

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.9/config.yml
```

![Image](/images/lab-setup/wazuh-platform/04-installation.png)

### Step 2 — Review the Configuration File

```bash
cat config.yml
```

![Image](/images/lab-setup/wazuh-platform/05-installation.png)

### Step 3 — Edit the Configuration File

Replace the placeholder IPs with Ubuntu Server static IP (192.168.48.129):

```bash
nano config.yml
```

![Image](/images/lab-setup/wazuh-platform/06-installation.png)

> **Important:** All three components run on the same server in this single-node deployment.

Save: `"Ctrl + O" > "Enter" > "Ctrl + X"`

---

## Part 2 — Generate Certificates

Wazuh uses SSL/TLS certificates for secure communication between components.
The installation assistant generates these automatically.

### Step 1 — Make the Script Executable

```bash
chmod +x wazuh-install.sh
```

![Image](/images/lab-setup/wazuh-platform/07-installation.png)

### Step 2 — Generate SSL Certificates

```bash
sudo bash wazuh-install.sh --generate-config-files
```

![Image](/images/lab-setup/wazuh-platform/08-installation.png)

---

## Part 3 — Install Wazuh Indexer

### Step 1 — Install Wazuh Indexer

```bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
```

![Image](/images/lab-setup/wazuh-platform/09-installation.png)

### Step 2 — Enable and Start Wazuh Indexer

```bash
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
```

![Image](/images/lab-setup/wazuh-platform/10-installation.png)

### Step 3 — Verify Wazuh Indexer is Running

```bash
sudo systemctl status wazuh-indexer
```

![Image](/images/lab-setup/wazuh-platform/11-installation.png)

---

## Part 4 — Install Wazuh Manager

### Step 1 — Install Wazuh Manager

```bash
sudo bash wazuh-install.sh --wazuh-server wazuh-1
```

![Image](/images/lab-setup/wazuh-platform/12-installation.png)

### Step 2 — Enable and Start Wazuh Manager

```bash
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
```

![Image](/images/lab-setup/wazuh-platform/13-installation.png)

### Step 3 — Verify Wazuh Manager is Running

```bash
sudo systemctl status wazuh-manager
```

![Image](/images/lab-setup/wazuh-platform/14-installation.png)

## Part 5 — Initialize Wazuh Indexer Cluster

This step must be completed before installing the Wazuh Dashboard.
It initializes the security settings required for the Dashboard to
connect to the Indexer.

### Step 1 — Initialize the Cluster

```bash
sudo bash wazuh-install.sh --start-cluster
```

![Image](/images/lab-setup/wazuh-platform/15-installation.png)

### Step 2 — Retrieve Admin Password

During installation, Wazuh auto-generates passwords for all users and
stores them inside `wazuh-install-files.tar`. Extract the password as follows:

```bash
sudo tar -O -xvf ~/wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

The output lists passwords for all Wazuh users. Look for the admin password:

![Image](/images/lab-setup/wazuh-platform/16-installation.png)


> **Important:** Copy this password and store it securely. It is required for:
> - Cluster health verification
> - Wazuh Dashboard login
> - Wazuh API access
> - Password recovery

### Step 3 — Verify Cluster Health

Replace `admin_password` with the admin password retrieved in Step 2:

```bash
curl -k -u admin:'admin_password' https://ubuntu_server_IP:9200/_cluster/health?pretty
```

![Image](/images/lab-setup/wazuh-platform/17-installation.png)

## Part 6 — Install Wazuh Dashboard

### Step 1 — Install Wazuh Dashboard

```bash
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```

![Image](/images/lab-setup/wazuh-platform/18-installation.png)

> The output also displays the admin password that we earlier retrieved manually.

### Step 2 — Enable and Start Wazuh Dashboard

```bash
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

![Image](/images/lab-setup/wazuh-platform/19-installation.png)

### Step 3 — Verify Wazuh Dashboard is Running

```bash
sudo systemctl status wazuh-dashboard
```

![Image](/images/lab-setup/wazuh-platform/20-installation.png)

---

## Part 7 — Verify All Services

### Step 1 — Check All Three Wazuh Services

```bash
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard
```

![Image](/images/lab-setup/wazuh-platform/21-installation.png)

### Step 2 — Check Listening Ports

Verify all seven Wazuh ports are listening:

```bash
sudo ss -tlnp | grep -E '443|1514|1515|1516|9200|9300|55000'
```

![Image](/images/lab-setup/wazuh-platform/22-installation.png)

> Output should show all seven ports listening.

### Step 3 — Verify UFW Allows All Wazuh Ports

```bash
sudo ufw status | grep -E '443|1514|1515|1516|9200|9300|55000'
```

![Image](/images/lab-setup/wazuh-platform/23-installation.png)

> Output should show all seven ports shown as ALLOW.

---

## Part 8 — Access the Wazuh Dashboard

### Step 1 — Access Dashboard from Host Machine Browser

Open a browser on **Windows 11 local host machine** and navigate to 

```
https://192.168.48.129
```

> Replace `192.168.48.129` with **ubuntu-server01** static IP.

### Step 2 — Accept SSL Certificate Warning

The browser will show a security warning because Wazuh uses a self-signed
certificate. Ignore the warning and proceed.

### Step 3 — Log In to Wazuh Dashboard

Use the retrieved `admin` credential and click **Log in**.

![Image](/images/lab-setup/wazuh-platform/24-installation.png)

The Wazuh Dashboard home page appears:

![Image](/images/lab-setup/wazuh-platform/25-installation.png)

Using the `HELP` button, we can see Wazuh version is **4.9.2**:

![Image](/images/lab-setup/wazuh-platform/26-installation.png)

---

## Part 9 — Take Snapshot

### Step 1 — Clean Up Before Snapshot

```bash
# Clear bash history
history -c && history -w

# Clear temporary files
sudo apt-get clean

#Exit SSH session
exit
```

![Image](/images/lab-setup/wazuh-platform/27-installation.png)

### Step 2 — Take VMware Snapshot

In VMware, with UBUNTU-SERVER01 VM running:

`VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `Wazuh Installed - Pre-Agent` |
| Description | `Wazuh 4.9.2 installed. Manager, Indexer, Dashboard all running. No agents connected yet.` |

Click **"Take Snapshot"**.

![Image](/images/lab-setup/wazuh-platform/28-installation.png)

> **Why snapshot with VM running?**
>
> Ubuntu Server runs Wazuh as a service continuously. Taking a snapshot
> while running preserves the fully operational state — restoring it
> brings all Wazuh services back up instantly without any manual
> intervention.

---
