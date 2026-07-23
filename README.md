# Purple Team Home Lab

Built to develop hands-on Purple Team skills alongside a background in
Detection Engineering and SOC operations. This lab demonstrates the full
Purple Team lifecycle — adversary emulation, ATT&CK-based detection validation
and coverage analysis.

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          HOST MACHINE                               │
│                    VMware Workstation Pro 17.6.4                    │
│                                                                     │
│  ┌──────────────────────────┐    ┌──────────────────────────────┐   │
│  │     WIN11-CLIENT01       │    │       UBUNTU-SERVER01        │   │
│  │     192.168.48.128       │    │       192.168.48.129         │   │
│  │  Windows 11 Ent. Eval    │    │   Ubuntu Server 22.04.5 LTS  │   │
│  │                          │    │                              │   │
│  │  ┌────────────────────┐  │    │  ┌────────────────────────┐  │   │
│  │  │  Sysmon 15.21      │  │    │  │  Wazuh 4.9.2           │  │   │
│  │  ├────────────────────┤  │    │  │  Manager + Indexer     │  │   │
│  │  │  Wazuh Agent 4.9.2 │  │    │  │  + Dashboard           │  │   │
│  │  ├────────────────────┤  │    │  ├────────────────────────┤  │   │
│  │  │  Atomic Red Team   │  │    │  │  MITRE Caldera 4.1.0   │  │   │
│  │  ├────────────────────┤  │    │  ├────────────────────────┤  │   │
│  │  │  Caldera Agent     │  │    │  │  ATT&CK Navigator      │  │   │
│  │  └────────────────────┘  │    │  └────────────────────────┘  │   │
│  │                          │    │                              │   │
│  └──────────────────────────┘    └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Lab Components

| Component | Version | Location | Role |
|---|---|---|---|
| VMware Workstation Pro | 17.6.4 | Host Machine | Virtualization platform |
| Windows 11 Enterprise Evaluation | Windows 11 | VMware | Attack target endpoint |
| Ubuntu Server | 22.04.5 LTS | VMware | Security monitoring & adversary emulation server |
| Wazuh | 4.9.2 | UBUNTU-SERVER01 | SIEM — log collection and detection |
| Sysmon | 15.21 | WIN11-CLIENT01 | Endpoint telemetry collection |
| Wazuh Agent | 4.9.2 | WIN11-CLIENT01 | Log forwarding to Wazuh Manager |
| ATT&CK Navigator | Latest | UBUNTU-SERVER01 | ATT&CK coverage visualization |
| Atomic Red Team | Latest | WIN11-CLIENT01 | Technique-level adversary emulation |
| MITRE Caldera | 4.1.0 | UBUNTU-SERVER01 | Adversary emulation platform |
| Sigma CLI | Latest | UBUNTU-SERVER01 | Detection rule conversion |

---

## Access Points

| Service | URL |
|---|---|
| Wazuh Dashboard | https://192.168.48.129 |
| ATT&CK Navigator | http://192.168.48.129:4200 |
| MITRE Caldera | http://192.168.48.129:8888 |

---

## Lab Setup Guides

| # | Component | Guide |
|---|---|---|
| 1 | VMware Workstation Pro 17.6.4 | [01-vmware](lab-setup/01-vmware/) |
| 2 | Windows 11 Enterprise Evaluation | [02-windows-11](lab-setup/02-windows-11/) |
| 3 | Ubuntu Server 22.04.5 LTS | [03-ubuntu-server](lab-setup/03-ubuntu-server/) |
| 4 | Wazuh 4.9.2 | [04-wazuh-platform](lab-setup/04-wazuh-platform/) |
| 5 | Sysmon 15.x | [05-sysmon](lab-setup/05-sysmon/) |
| 6 | Wazuh Agent 4.9.2 | [06-wazuh-agent](lab-setup/06-wazuh-agent/) |
| 7 | ATT&CK Navigator | [07-attack-navigator](lab-setup/07-attack-navigator/) |
| 8 | Atomic Red Team | [08-atomic-red-team](lab-setup/08-atomic-red-team/) |
| 9 | MITRE Caldera 4.1.0 | [09-caldera](lab-setup/09-caldera/) |

---
