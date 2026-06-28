# MITRE ATT&CK Navigator — Installation & Configuration Guide

## Overview

MITRE ATT&CK Navigator is a web-based tool for annotating and exploring
the MITRE ATT&CK framework matrix. It allows Purple Team practitioners
to visualize, track, and communicate detection coverage across ATT&CK
tactics and techniques.

### Role in Purple Team Lab

| Function | Description |
|---|---|
| **Coverage Visualization** | Color-code techniques by detection status — detected, not detected, or blocked |
| **Gap Analysis** | Identify which techniques have no detection coverage |
| **Reporting** | Export heat maps for inclusion in Purple Team assessment reports |

---

## Prerequisites

### Verify Prerequisites on ubuntu-server01

SSH into ubuntu-server01:

```powershell
ssh labadmin@192.168.48.129
```

Verify Ubuntu version:

```bash
lsb_release -a
```

![Image](/images/lab-setup/attack-navigator/01-pre-requisite.png)

Verify UFW allows port 4200:

```bash
sudo ufw status | grep 4200
```

![Image](/images/lab-setup/attack-navigator/02-pre-requisite.png)

If missing, run below commands to allow the port:

```bash
sudo ufw allow 4200/tcp
sudo ufw reload
```

---

## Part 1 — Install Dependencies

### Step 1 — Install Node.js

ATT&CK Navigator requires Node.js v18.x or higher.

```bash
# Add NodeSource repository for Node.js 20.x LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

![Image](/images/lab-setup/attack-navigator/03-installation.png)

![Image](/images/lab-setup/attack-navigator/04-installation.png)

Install Node.js:
```bash
sudo apt-get install -y nodejs
```

![Image](/images/lab-setup/attack-navigator/05-installation.png)

Verify installation:

```bash
node --version
npm --version
```

![Image](/images/lab-setup/attack-navigator/06-installation.png)

### Step 2 — Install Git

```bash
# Verify git is installed
git --version

# If not installed
sudo apt-get install -y git
```

![Image](/images/lab-setup/attack-navigator/07-installation.png)

---

## Part 2 — Install ATT&CK Navigator

### Step 1 — Clone ATT&CK Navigator Repository

```bash
# Create directory under /opt
sudo mkdir -p /opt/attack-navigator

# Set ownership to labadmin so we can manage it without sudo
sudo chown -R labadmin:labadmin /opt/attack-navigator

# Clone into /opt
git clone https://github.com/mitre-attack/attack-navigator.git /opt/attack-navigator

# Verify clone succeeded
ls -la /opt/attack-navigator
```

![Image](/images/lab-setup/attack-navigator/08-installation.png)

![Image](/images/lab-setup/attack-navigator/09-installation.png)

### Step 2 — Navigate to Application Directory

```bash
# Navigate to application directory
cd /opt/attack-navigator/nav-app

# Verify contents
ls -la
```

![Image](/images/lab-setup/attack-navigator/10-installation.png)

### Step 3 — Install Node Dependencies

```bash
npm install
```

![Image](/images/lab-setup/attack-navigator/11-installation.png)

Watch for successful completion: `added XXXX packages, and audited XXXX packages in Xs`

> **Note:** We may see `npm warn deprecated` messages during installation.
> These are warnings about older sub-dependencies and do not affect
> functionality. They can be safely ignored.

---

## Part 3 — Host ATT&CK Navigator

### Step 1 — Build the Application

Converting the ATT&CK Navigator source code into a production-ready web application that can be served by a web server and accessed through a browser.

```bash
# Build production version
npm run build
```

![Image](/images/lab-setup/attack-navigator/12-installation.png)

Verify build output exists:
```bash
ls dist/browser/
```

![Image](/images/lab-setup/attack-navigator/12-installation-01.png)

> **Build once, serve forever.** The build only needs to be re-run when
> ATT&CK Navigator itself is updated. For day-to-day lab use, the built
> files are served directly — no recompilation needed.

### Step 2 — Install http-server

`http-server` is a lightweight static file server — it simply serves
the pre-built files from the dist/ directory:

```bash
# Install http-server
sudo npm install -g http-server

# Verify installation
http-server --version
```

![Image](/images/lab-setup/attack-navigator/13-installation.png)

### Step 3 — Test http-server Manually

Before setting up PM2, verify http-server works correctly:

```bash
http-server /opt/attack-navigator/nav-app/dist/browser --port 4200 --host 0.0.0.0 --cors --silent
```

> **Note:** This command runs in the foreground. The terminal will be
> occupied while Navigator is running.

![Image](/images/lab-setup/attack-navigator/14-installation.png)

Open browser on local Windows 11 host and browse below to access ATT&CK Navigator:

```
http://192.168.48.129:4200
```

![Image](/images/lab-setup/attack-navigator/15-installation.png)

Before proceeding, press **Ctrl + C** to stop the manually running http-sever.

---

## Part 4 — Run Navigator as Background Service

Running Navigator as a background service means it starts automatically
and does not require a terminal session to remain open.

### Step 1 — Install PM2 Process Manager

PM2 manages http-server process as a background service:

```bash
sudo npm install -g pm2
```

Verify:

```bash
pm2 --version
```

![Image](/images/lab-setup/attack-navigator/16-installation.png)

---

### Step 2 — Create Navigator Start Script

```bash
# Create the start script inside the application directory
sudo tee /opt/attack-navigator/start-navigator.sh > /dev/null << 'EOF'
#!/bin/bash
http-server /opt/attack-navigator/nav-app/dist/browser --port 4200 --host 0.0.0.0 --cors --silent
EOF

# Give executable permission
sudo chmod +x /opt/attack-navigator/start-navigator.sh

# Set ownership to labadmin
sudo chown labadmin:labadmin /opt/attack-navigator/start-navigator.sh
```

Verify:

```bash
ls -l /opt/attack-navigator/start-navigator.sh
cat /opt/attack-navigator/start-navigator.sh
```

![Image](/images/lab-setup/attack-navigator/17-installation.png)

---

### Step 3 — Start Navigator with PM2

```bash
pm2 start /opt/attack-navigator/start-navigator.sh --name "attack-navigator"
```

![Image](/images/lab-setup/attack-navigator/18-installation.png)

### Step 4 — Configure PM2 to Start on Boot

Generate startup script:

```bash
pm2 startup
```

Copy and run the exact command shown in terminal output of above command.

Save the process list:
```bash
pm2 save
```

![Image](/images/lab-setup/attack-navigator/19-installation.png)

![Image](/images/lab-setup/attack-navigator/20-installation.png)

---

### Step 5 — Verify Full Setup

```bash
# Check PM2 status
pm2 status
```

```bash
# Check port is listening
sudo ss -tlnp | grep 4200
```

![Image](/images/lab-setup/attack-navigator/21-installation.png)

Verify if ATT&CK Navigator is still accessible via browser:

```
http://192.168.48.129:4200
```

---

### Step 6 — Verify Navigator Survives Reboot

```bash
sudo reboot
```

![Image](/images/lab-setup/attack-navigator/22-installation.png)

After reboot, SSH back in:

```bash
pm2 status
sudo ss -tlnp | grep 4200
```

Browse to `http://192.168.48.129:4200`. Navigator should load without any manual intervention.

![Image](/images/lab-setup/attack-navigator/23-installation.png)

---

### When to Re-run npm run build

We only need to rebuild when updating ATT&CK Navigator itself:

```bash
# Update Navigator to latest version
cd /opt/attack-navigator
git pull

# Rebuild
cd nav-app
npm install
npm run build

# Restart service
pm2 restart attack-navigator
```

For day-to-day lab use — creating layers, exporting JSON,
validating detections — no rebuild is ever needed.

---

## Part 5 — Access ATT&CK Navigator

### Step 1 — Access from Windows 11 Host Browser

Open a browser on Windows 11 host machine and navigate to:

```
http://192.168.48.129:4200
```

> **Note:** ATT&CK Navigator uses HTTP — not HTTPS. No certificate warning will appear.

### Step 2 — Verify ATT&CK Data Loads

On ATT&CK Navigator page,
- Click "Create New Layer"
- Under `More Options`,
	- Select latest version.
	- Select "Enterprise ATT&CK" as domain.
	- Click "Create layer from version".

![Image](/images/lab-setup/attack-navigator/24-installation.png)

The full ATT&CK Enterprise matrix loads with all tactics as column headers
and techniques as rows:

![Image](/images/lab-setup/attack-navigator/25-installation.png)

---

## Part 6 — Creating ATT&CK Layers

### Step 1 — Create a Purple Team Coverage Baseline Layer

This layer represents our current detection coverage baseline.

**Step 1.1 — Create new layer:**

```
ATT&CK Navigator home
- Click "Create New Layer"
- Select domain as "Enterprise ATT&CK"
- Select current version
- Click "Create"
```

![Image](/images/lab-setup/attack-navigator/24-installation.png)

**Step 1.2 — Name the layer:**

```
Click the layer name tab at the top
- Click "Layer Controls" tab
- Click "layer settings" gear icon
- Name: "Purple Team Coverage - Baseline"
- Description: "Initial coverage baseline before Purple Team testing"
- Click "Close"
```

![Image](/images/lab-setup/attack-navigator/26-installation.png)

**Step 1.3 — Understand the color scheme:**

For consistent documentation use this color convention:

| Color | Meaning |
|---|---|
| Dark Green | Validated — Detected and Alerted |
| Light Green | Detected — No Alert (telemetry only) |
| Yellow | Partially Covered — needs improvement |
| Red | Not Detected — critical gap |
| White (no color) | Not Tested |

**Step 1.4 — Color a technique:**

```
- Click on "Technique Controls" tab.
- Select a technique (e.g., T1059.001 PowerShell)
- Click the color swatch
- Select dark green color
- Add comment: "Validated 2026-06-21. Wazuh alert fires on execution."
```

![Image](/images/lab-setup/attack-navigator/27-installation.png)

![Image](/images/lab-setup/attack-navigator/28-installation.png)


### Step 2 — Score Techniques

Scores allow quantitative coverage analysis:

```
- Click a technique
- Go to "Technique Controls" tab.
- Click on Score swatch
- Enter 1–5
```

![Image](/images/lab-setup/attack-navigator/29-installation.png)

Use this scoring convention:

| Score | Meaning |
|---|---|
| 5 | Validated — Detected + Alerted + High Fidelity |
| 4 | Detected + Alerted — alert quality needs improvement |
| 3 | Detected — No Alert (telemetry present) |
| 2 | Partial Visibility — insufficient telemetry |
| 1 | No Visibility |
| 0 | Not Tested |

---

## Part 7 — Mapping Sysmon Detections to ATT&CK

### Step 1 — Map Sysmon Event IDs to Techniques

Use this reference table to map Sysmon telemetry to ATT&CK techniques
and color them in Navigator:

| Sysmon EID | ATT&CK Technique | Technique ID | Color if Detected |
|---|---|---|---|
| EID 1 — Process Create | Command and Scripting Interpreter | T1059 | Green |
| EID 1 — PowerShell | PowerShell | T1059.001 | Green |
| EID 1 — cmd.exe | Windows Command Shell | T1059.003 | Green |
| EID 3 — Network Connection | Application Layer Protocol | T1071 | Green |
| EID 7 — Image Loaded | Shared Modules | T1129 | Green |
| EID 8 — CreateRemoteThread | Process Injection | T1055 | Green |
| EID 10 — ProcessAccess (LSASS) | OS Credential Dumping: LSASS | T1003.001 | Green |
| EID 11 — FileCreate | Ingress Tool Transfer | T1105 | Green |
| EID 13 — RegistryValueSet (Run) | Registry Run Keys | T1547.001 | Green |
| EID 22 — DNS Query | DNS | T1071.004 | Green |

**How to apply in Navigator:**

For each Sysmon EID that generates events in Wazuh:

```
1. Find the corresponding technique in the matrix
2. Click the technique
3. Set color to green — confirmed telemetry exists
4. Add comment with: "Sysmon EID X detected. Wazuh alert: [rule name]"
5. Set score: 5 if alert fires, 3 if telemetry only
```

### Step 2 — Validate Sysmon Coverage Procedure

For each technique we want to validate:

**Step 2.1 — Check if Sysmon captures it:**

```powershell
# On WIN11-CLIENT01
# Example: checking T1059.001 PowerShell coverage
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | Where-Object { $_.Id -eq 1 } | Where-Object { $_.Message -like "*powershell*" } | Select-Object -First 3 TimeCreated, Message
```

**Step 2.2 — Check if Wazuh alerts on it:**

```bash
# On ubuntu-server01
sudo grep -i "powershell" /var/ossec/logs/alerts/alerts.log | tail -5
```

**Step 2.3 — Color in Navigator based on result:**

```
Both Sysmon + Wazuh alert	: Score 5, Color dark green
Sysmon only, no alert		: Score 3, Color yellow
Neither						: Score 1, Color red
```

---

## Part 8 — Export and Import Layers

### Step 1 — Export a Layer as JSON

In ATT&CK Navigator with layer open
- Click on "Layer Controls"
- Click the download icon in the toolbar
- Select "Download as JSON"

![Image](/images/lab-setup/attack-navigator/29-installation-01.png)

The file saves as `layer.json`. Rename it descriptively: `purple-team-coverage-baseline-2026-06-27.json`

### Step 2 — Export Layer as SVG Image

In ATT&CK Navigator with layer open
- Click on "Layer Controls"
- Click the download icon in the toolbar
- Select "Download as SVG"

This creates a visual heat map image for inclusion in reports.

Rename to: `coverage-heatmap-2026-06-21.svg`

### Step 3 — Import an Existing Layer

ATT&CK Navigator home
- Click "Open Existing Layer"
- Select "Upload from local"
- Browse to .json layer file
- Click Open

![Image](/images/lab-setup/attack-navigator/29-installation-02.png)

The layer loads with all previous colors and annotations preserved.

---

## Part 9 — Mapping Wazuh Rules to ATT&CK

### Step 1 — Find Wazuh Rules with ATT&CK Mapping

Wazuh includes built-in rules that already map to ATT&CK:

```bash
# On ubuntu-server01 — find rules with MITRE mapping
sudo grep -Rl "<mitre>" /var/ossec/ruleset/rules/ | wc -l
sudo grep -Rl "<mitre>" /var/ossec/ruleset/rules/ | head
```

![Image](/images/lab-setup/attack-navigator/30-installation.png)

```bash
# On ubuntu-server01 — find rules containing a specific ATT&CK technique
sudo grep -Rl "T1059.001" /var/ossec/ruleset/rules/ | wc -l
sudo grep -Rl "T1059.001" /var/ossec/ruleset/rules/ | head
```

![Image](/images/lab-setup/attack-navigator/31-installation.png)

```bash
# Find Sysmon-specific rules
sudo grep -Rl "sysmon" /var/ossec/ruleset/rules/ | wc -l
sudo grep -Rl "sysmon" /var/ossec/ruleset/rules/ | head
```

![Image](/images/lab-setup/attack-navigator/32-installation.png)

```bash
# View a specific Sysmon rule file — based on rule filename
sudo cat /var/ossec/ruleset/rules/0800-sysmon_id_1.xml | head -20
```

![Image](/images/lab-setup/attack-navigator/33-installation.png)

```bash
# View a specific Sysmon rule file — based on rule id (e.g. 92203)
sudo grep -Rl 'rule id="92203"' /var/ossec/ruleset/rules/
sudo awk '/id="92203"/,/<\/rule>/' /var/ossec/ruleset/rules/0830-sysmon_id_11.xml
```

![Image](/images/lab-setup/attack-navigator/34-installation.png)

### Step 2 — Extract All Wazuh ATT&CK Mappings

```bash
# Extract all technique IDs from Wazuh rules
sudo grep -r "<id>T[0-9]" /var/ossec/ruleset/rules/ | grep -o "T[0-9]\{4\}\(\.[0-9]\{3\}\)\?" | sort -u
```

![Image](/images/lab-setup/attack-navigator/35-installation.png)

This gives us the complete list of ATT&CK techniques that Wazuh rules
cover. Each technique in this list can be colored in Navigator.

### Step 3 — Create Wazuh Coverage Layer

Perform below steps on **ubuntu-server01**:-

**Step 3.1 — Generate Wazuh coverage report:**

Create the script `export_wazuh_attack_mapping.py` that maps MITRE Techniques to related Wazuh Rule IDs:

```bash
python3 --version
nano export_wazuh_attack_mapping.py
```

![Image](/images/lab-setup/attack-navigator/36-installation.png)

Paste below Python script into the file and save it:

```python
#!/usr/bin/env python3

"""
Generate a CSV mapping "MITRE ATT&CK Techniques" to "Wazuh Rule IDs".

This script:
- Parses all Wazuh XML rule files.
- Extracts MITRE ATT&CK Technique IDs.
- Groups multiple Wazuh Rule IDs under the same ATT&CK Technique.
- Exports the results to a CSV file.
"""

import os
import csv
import glob
import re
import xml.etree.ElementTree as ET
from collections import defaultdict

# ==========================================================
# Configuration
# ==========================================================

RULES_PATH = "/var/ossec/ruleset/rules"
OUTPUT_DIR = "output"
OUTPUT_FILE = os.path.join(OUTPUT_DIR, "wazuh_mitre_attack_mapping.csv")

# Valid MITRE ATT&CK Technique format:
# Examples:
#   T1059
#   T1003.001
MITRE_PATTERN = re.compile(r"^T\d{4}(\.\d{3})?$")


def main():

    # ======================================================
    # Create output directory
    # ======================================================

    os.makedirs(OUTPUT_DIR, exist_ok=True)

    # ======================================================
    # Dictionary:
    # Key   = Technique ID
    # Value = Set of Wazuh Rule IDs
    # ======================================================

    attack_mapping = defaultdict(set)

    # ======================================================
    # Read XML rule files
    # ======================================================

    xml_files = sorted(glob.glob(os.path.join(RULES_PATH, "*.xml")))

    print(f"[+] Found {len(xml_files)} XML rule files.\n")

    for xml_file in xml_files:

        try:
            with open(xml_file, "r", encoding="utf-8") as f:
                xml_data = f.read()

            # Wazuh rule files may contain multiple top-level
            # <group> elements, which is not valid XML for
            # ElementTree. Wrap the content in a temporary
            # root element before parsing.
            xml_data = f"<root>\n{xml_data}\n</root>"

            root = ET.fromstring(xml_data)

        except ET.ParseError as e:
            print(f"[!] Skipping invalid XML: {xml_file}")
            print(f"    {e}\n")
            continue

        except Exception as e:
            print(f"[!] Error reading {xml_file}")
            print(f"    {e}\n")
            continue

        # --------------------------------------------------
        # Process every Wazuh rule
        # --------------------------------------------------

        for rule in root.iter("rule"):

            rule_id = rule.get("id")

            if not rule_id:
                continue

            mitre = rule.find("mitre")

            if mitre is None:
                continue

            for technique in mitre.findall("id"):

                if not technique.text:
                    continue

                technique_id = technique.text.strip()

                # Ignore variables such as:
                # $(threat.software.id)
                if technique_id.startswith("$("):
                    continue

                # Ignore anything that is not a valid
                # MITRE ATT&CK Technique ID
                if not MITRE_PATTERN.match(technique_id):
                    continue

                attack_mapping[technique_id].add(rule_id)

    # ======================================================
    # Write CSV
    # ======================================================

    with open(OUTPUT_FILE, "w", newline="", encoding="utf-8") as csvfile:

        writer = csv.writer(csvfile)

        writer.writerow([
            "Technique",
            "Rule IDs"
        ])

        for technique in sorted(attack_mapping.keys()):

            rule_ids = sorted(
                attack_mapping[technique],
                key=int
            )

            writer.writerow([
                technique,
                ", ".join(rule_ids)
            ])

    # ======================================================
    # Summary
    # ======================================================

    print("==============================================")
    print(" Wazuh ATT&CK Mapping Generated Successfully")
    print("==============================================")
    print(f"Unique ATT&CK Techniques : {len(attack_mapping)}")
    print(f"Output CSV               : {OUTPUT_FILE}")
    print("==============================================")


if __name__ == "__main__":
    main()
```

Run the script to generate CSV:
```bash
sudo python3 export_wazuh_attack_mapping.py
```

Verify the content of output CSV:

```bash
head output/wazuh_mitre_attack_mapping.csv
```

![Image](/images/lab-setup/attack-navigator/37-installation.png)

**Step 3.2 — Generate ATT&CK Navigator layer JSON:**

Create the script `generate_attack_navigator_layer.py` that generates layer JSON file:

```bash
nano generate_attack_navigator_layer.py
```

Paste below Python script into the file and save it:

```python
#!/usr/bin/env python3

"""
Generate an ATT&CK Navigator Layer JSON from a CSV file.

Input CSV:
------------
Technique,Rule IDs
T1003.001,"92023, 92024"
T1012,"91808, 91836"

Output:
------------
wazuh_attack_layer.json

Each technique will:
- be colored GREEN
- have Score = 100
- contain the Wazuh Rule IDs as a formatted comment
"""

import csv
import json

# ==========================================================
# Configuration
# ==========================================================

INPUT_CSV = "output/wazuh_mitre_attack_mapping.csv"
OUTPUT_JSON = "output/wazuh_attack_layer.json"

GREEN = "#31a354"

# ==========================================================
# Read CSV
# ==========================================================

techniques = []

with open(INPUT_CSV, newline="", encoding="utf-8") as csvfile:

    reader = csv.DictReader(csvfile)

    for row in reader:

        technique = row["Technique"].strip()
        rule_ids = row["Rule IDs"].strip()

        # Convert comma-separated Rule IDs into a list
        rule_list = [r.strip() for r in rule_ids.split(",") if r.strip()]

        # Build formatted comment
        comment = (
            f"Wazuh Detection Coverage\n\n"
            f"Coverage: {len(rule_list)} rule(s)\n\n"
            f"Rule IDs:\n"
            + "\n".join(rule_list)
        )

        techniques.append(
            {
                "techniqueID": technique,
                "score": 100,
                "color": GREEN,
                "comment": comment,
                "enabled": True,
                "metadata": [],
                "links": [],
                "showSubtechniques": False
            }
        )

# ==========================================================
# Build ATT&CK Navigator Layer
# ==========================================================

layer = {

    "name": "Wazuh ATT&CK Coverage",

    "versions": {
        "attack": "19",
        "navigator": "5.3.2",
        "layer": "4.5"
    },

    "domain": "enterprise-attack",

    "description": "Automatically generated from Wazuh MITRE ATT&CK mappings.",

    "filters": {
        "platforms": [
            "Windows",
            "Linux",
            "macOS",
            "Containers",
            "Network Devices",
            "Office Suite",
            "Identity Provider",
            "IaaS",
            "SaaS",
            "PRE",
            "ESXi"
        ]
    },

    "sorting": 0,

    "layout": {
        "layout": "side",
        "aggregateFunction": "average",
        "showID": False,
        "showName": True,
        "showAggregateScores": False,
        "countUnscored": False,
        "expandedSubtechniques": "none"
    },

    "hideDisabled": False,

    "techniques": techniques,

    "gradient": {
        "colors": [
            "#ff6666ff",
            "#ffe766ff",
            "#8ec843ff"
        ],
        "minValue": 0,
        "maxValue": 100
    },

    "legendItems": [
        {
            "label": "Covered by Wazuh",
            "color": GREEN
        }
    ],

    "metadata": [],
    "links": [],

    "showTacticRowBackground": False,
    "tacticRowBackground": "#dddddd",

    "selectTechniquesAcrossTactics": True,
    "selectSubtechniquesWithParent": False,
    "selectVisibleTechniques": False
}

# ==========================================================
# Write JSON
# ==========================================================

with open(OUTPUT_JSON, "w", encoding="utf-8") as outfile:
    json.dump(layer, outfile, indent=4)

# ==========================================================
# Summary
# ==========================================================

print("==============================================")
print(" ATT&CK Navigator Layer Generated")
print("==============================================")
print(f"Techniques exported : {len(techniques)}")
print(f"Output file         : {OUTPUT_JSON}")
print("==============================================")
```

Run the script:
```bash
python3 generate_attack_navigator_layer.py
```

![Image](/images/lab-setup/attack-navigator/38-installation.png)

**Step 3.3 — Import generated layer in ATT&CK Navigator:**

`Home > "Open Existing Layer" > "Upload from local"`

![Image](/images/lab-setup/attack-navigator/39-installation.png)
 
---

## Part 10 — Building Assessment Layers

### Step 1 — Create a Post-Exercise Layer

After completing an adversary emulation exercise, update layer:

**Example: After running T1059.001 PowerShell atomic test:**

1. Open coverage layer in Navigator
2. Find T1059.001 in the matrix
3. Click the technique
4. Set color: green — alert fired in Wazuh
5. Set score: `5`
6. Comment: `Validated 2026-06-21. Atomic test T1059.001. Wazuh alert: rule.id 92002 fired. Sysmon EID 1 + EID 4104 captured.`
7. Export updated layer

---

## Part 11 — Take Snapshot

### Step 1 — Take Snapshot After Navigator Installation

```bash
# On ubuntu-server01 — clean up before snapshot
history -c && history -w
sudo apt-get clean
```

![Image](/images/lab-setup/attack-navigator/40-installation.png)

In VMware: `VM menu > Snapshot > Take Snapshot`

| Field | Value |
|---|---|
| Name | `ATT&CK Navigator Installed` |
| Description | `Wazuh running. ATT&CK Navigator installed and running on port 4200` |

![Image](/images/lab-setup/attack-navigator/41-installation.png)

---
