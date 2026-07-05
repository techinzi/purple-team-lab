# Lab Setup

Step-by-step installation and configuration guides for all components
of the Purple Team home lab environment.

---

## Lab Components

| # | Component | Version | Guide |
|---|---|---|---|
| 1 | VMware Workstation Pro | 17.6.4 | [01-vmware](01-vmware/) |
| 2 | Windows 11 Enterprise Evaluation | Windows 11 | [02-windows-11](02-windows-11/) |
| 3 | Ubuntu Server | 22.04.5 LTS | [03-ubuntu-server](03-ubuntu-server/) |
| 4 | Wazuh Platform | 4.9.2 | [04-wazuh-platform](04-wazuh-platform/) |
| 5 | Sysmon | 15.21 | [05-sysmon](05-sysmon/) |
| 6 | Wazuh Agent | 4.9.2 | [06-wazuh-agent](06-wazuh-agent/) |
| 7 | ATT&CK Navigator | Latest | [07-attack-navigator](07-attack-navigator/) |
| 8 | Atomic Red Team | Latest | [08-atomic-red-team](08-atomic-red-team/) |
| 9 | MITRE Caldera | 4.1.0 | [09-caldera](09-caldera/) |

---

## Build Order

The lab is recommended to be built in the sequence given above because
some component depends on the previous one to be operational.

---

## VM Specifications

| VM | OS | IP Address | vCPU | RAM | Disk |
|---|---|---|---|---|---|
| WIN11-CLIENT01 | Windows 11 Enterprise Eval | 192.168.48.128 | 2 | 4 GB | 80 GB |
| UBUNTU-SERVER01 | Ubuntu Server 22.04.5 LTS | 192.168.48.129 | 2 | 4 GB | 80 GB |

---
