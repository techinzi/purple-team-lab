# Windows 11 Enterprise Evaluation - Download & Installation Guide

## Overview

This guide covers the complete download and installation of Windows 11 Enterprise Evaluation inside a VMware Workstation Pro 17.

> **Evaluation Period:** Windows 11 Enterprise Evaluation is free for 90 days. No product key is required during installation.

---

## Part 1 - Download Windows 11 Enterprise Evaluation ISO

### Step 1 - Navigate to Microsoft Evaluation Center

Browse to the [Microsoft Evaluation Center](https://www.microsoft.com/en-in/evalcenter) and click on the **Windows** tab at the top and select **Windows 11 Enterprise**:

![Evaluation Center](/images/lab-setup/windows-11/01-evaluation-center.png)

---

### Step 2 - Select Download Option

Under **"Get started for free"** click on **"Download the ISO - Windows 11
Enterprise"**.

![Select Product](/images/lab-setup/windows-11/02-select-product.png)

---

### Step 3 - Complete Registration

Fill in the registration details and click **"Download now"**.

![Registration](/images/lab-setup/windows-11/03-registration.png)

---

### Step 4 - Download ISO

Select **English (United States) 64-bit edition** for the ISO download.

![Download](/images/lab-setup/windows-11/04-download.png)

> **Note:** The ISO can be deleted after installation is complete and the initial snapshot has been taken and verified. If needed again, it can be re-downloaded free from the Microsoft Evaluation Center.

---

## Part 2 - Create a New Virtual Machine in VMware Workstation Pro 17

### Step 1 - Open New Virtual Machine Wizard

Go to **File > New Virtual Machine** to open the New Virtual Machine Wizard.

![Open New VM Wizard](/images/lab-setup/windows-11/05-installation-01.png)

---

### Step 2 - Select Configuration Type

Select **"Custom (advanced)"** and click **Next**.

![Configuration Type](/images/lab-setup/windows-11/06-installation-02.png)

---

### Step 3 - Hardware Compatibility

Select **Hardware compatibility** as **Workstation 17.x** and click **Next**.

![Hardware Compatibility](/images/lab-setup/windows-11/07-installation-03.png)

---

### Step 4 - Guest OS Installation

Select **"I will install the operating system later"** and click **Next**.

![Install Later](/images/lab-setup/windows-11/08-installation-04.png)

---

### Step 5 - Select Guest Operating System

| Setting | Value |
|---|---|
| Guest operating system | Microsoft Windows |
| Version | Windows 11 x64 |

Click **Next**.

![Select Guest OS](/images/lab-setup/windows-11/09-installation-05.png)

> **Note:** Selecting Windows 11 x64 automatically configures UEFI firmware and adds TPM 2.0 by default. Do not select Windows 10 x64.

---

### Step 6 - Set VM Name and Location

| Setting | Value |
|---|---|
| Virtual machine name | `WIN11-CLIENT01` |
| Location | `C:\VirtualMachines\WIN11-CLIENT01` |

> **Note:** Ensure the location drive has at least 80 GB free space.
> "VirtualMachines" directory needs to be created before specifying the VM location.

Click **Next**.

![VM Name](/images/lab-setup/windows-11/10-installation-06.png)

---

### Step 7 - Encryption for TPM Support

VMware requires VM encryption before a virtual TPM can be added.

| Setting | Value |
|---|---|
| Encryption type | Only the files needed to support a TPM are encrypted |
| Remember password | Unchecked |

> **Important:** Store this encryption password securely. If lost, the VM cannot be recovered.

Click **Next**.

![Encryption](/images/lab-setup/windows-11/11-installation-07.png)

---

### Step 8 - Firmware Type

| Setting | Value |
|---|---|
| Firmware type | UEFI |
| Secure Boot | Enabled |

Click **Next**.

![Firmware](/images/lab-setup/windows-11/12-installation-08.png)

---

### Step 9 - Processor Configuration

| Setting | Value |
|---|---|
| Number of processors | 1 |
| Number of cores per processor | 2 |

Click **Next**.

![Processor](/images/lab-setup/windows-11/13-installation-09.png)

---

### Step 10 - Memory

Assign **4096 MB** memory and click **Next**.

![Memory](/images/lab-setup/windows-11/14-installation-10.png)

> **Note:** Never allocate more than 50% of host RAM across all running VMs.

---

### Step 11 - Network Type

Select **NAT** and click **Next**.

![Network](/images/lab-setup/windows-11/15-installation-11.png)

---

### Step 12 - I/O Controller Type

Select **LSI Logic SAS (Recommended)** and click **Next**.

![Controller](/images/lab-setup/windows-11/16-installation-12.png)

---

### Step 13 - Disk Type

Select **NVMe (Recommended)** and click **Next**.

![Image](/images/lab-setup/windows-11/16-installation-12-1.png)

---

### Step 14 - Select a Disk

Select **"Create a new virtual disk"** and click **Next**.

![Image](/images/lab-setup/windows-11/16-installation-12-2.png)

---

### Step 15 - Disk Capacity

| Setting | Value |
|---|---|
| Maximum disk size | 80 GB |
| Allocate all disk space now | Unchecked |
| Disk storage | Store virtual disk as a single file |

Click **Next**.

![Disk Capacity](/images/lab-setup/windows-11/17-installation-13.png)

> Virtual disks are dynamically allocated — the `.vmdk` file on the host only
> grows as data is written inside the VM. 80 GB is the maximum ceiling, not
> the immediate disk usage.

---

### Step 16 - Disk File Name

Leave the default filename as suggested by VMware and click **Next**.

![Disk File Name](/images/lab-setup/windows-11/18-installation-14.png)

---

### Step 17 - Review Summary

Before finishing, click **"Customize Hardware"** to configure display, ISO,
and USB settings covered in Part 3.

![Review Summary](/images/lab-setup/windows-11/19-installation-15.png)

---

## Part 3 - Configure VM Hardware Settings

All steps below are performed inside the **Customize Hardware** window.

### Step 1 - Configure Display

Click **Display** in the left panel:

| Setting | Value |
|---|---|
| Accelerate 3D graphics | Enabled |
| Monitors | Use host setting for monitors |
| Graphics memory | 4 GB |
| Display scaling | Automatically adjust |

![Image](/images/lab-setup/windows-11/20-installation-16.png)

> Virtual graphics memory is drawn from host RAM, not a dedicated GPU. Allocate based on host machine's physical RAM:
>
> For host with 16GB of RAM, allocate 4 GB for "Graphics memory" to leave headroom for host OS and other VMs
>
> For host with 32GB of RAM or more, allocate 8 GB for "Graphics memory".

---

### Step 2 - Attach Windows 11 ISO

Click **New CD/DVD (SATA)** in the left panel:

| Setting | Value |
|---|---|
| Connect at power on | Enabled |
| Connection | Use ISO image file |
| ISO path | Browse to downloaded ISO file |

![Image](/images/lab-setup/windows-11/21-installation-17.png)

---

### Step 3 - Configure USB Controller

Click **USB Controller** in the left panel:

| Setting | Value |
|---|---|
| USB compatibility | USB 3.1 |
| Show all USB input devices | Unchecked |

![Image](/images/lab-setup/windows-11/22-installation-18.png)

---

### Step 4 - Finish VM Creation

Click **Close** to return to the summary screen.

![Image](/images/lab-setup/windows-11/23-installation-19.png)

Click **Finish** to create the virtual machine.

![Image](/images/lab-setup/windows-11/24-installation-20.png)

> **Note:** VMware Workstation Pro 17 automatically adds TPM 2.0 when Windows 11 x64 is selected as the guest OS.
> 
> Verify by checking **VM Settings > Hardware** - **Trusted Platform Module** should already appear in the hardware list.

![Image](/images/lab-setup/windows-11/25-installation-21.png)

---

## Part 4 - Boot and Install Windows 11

### Step 1 - Power On the VM

Click **"Power on this virtual machine"** in VMware.

![Image](/images/lab-setup/windows-11/28-installation-24.png)

---

### Step 2 - Boot from ISO

When the below prompt appears, immediately click inside the VM window and press any key:

```Press any key to boot from CD or DVD...```

> **NOTE:** This prompt appears for only few seconds. If missed, the VM will attempt to boot from the virtual hard disk (which is empty) and display an error. Simply restart the VM and try again.

---

### Step 3 - Windows Setup

Select the following options:

| Setting | Value |
|---|---|
| Language to install | English (United States) |
| Time and currency format | English (United States) |
| Keyboard or input method | US |

![Image](/images/lab-setup/windows-11/29-installation-25.png)
![Image](/images/lab-setup/windows-11/30-installation-26.png)

Click **"Install Windows 11"**, check **"I agree"** and click **Next**.

![Image](/images/lab-setup/windows-11/31-installation-27.png)

---

### Step 4 - Accept License Terms

Click **Accept**.

![Image](/images/lab-setup/windows-11/32-installation-28.png)

---

### Step 5 - Select Installation Drive

Select **Drive 0 Unallocated Space** and click **Next**.

![Image](/images/lab-setup/windows-11/33-installation-29.png)

Click **Install**

![Image](/images/lab-setup/windows-11/34-installation-30.png)

> **Note:** During installation the VM restarts automatically multiple times.
> 
> Do not press any key when **"Press any key to boot from CD or DVD"** appears during restarts - let the VM boot from the hard disk automatically.

![Image](/images/lab-setup/windows-11/35-installation-31.png)

---

### Step 6 - Initial Setup (OOBE)

**Region and Keyboard:**

| Prompt | Selection |
|---|---|
| Country | India |
| Keyboard layout | US |
| Second keyboard layout | Skip |

![Image](/images/lab-setup/windows-11/37-installation-33.png)
![Image](/images/lab-setup/windows-11/38-installation-34.png)
![Image](/images/lab-setup/windows-11/39-installation-35.png)

---

### Step 7 - Create Local Account

When prompted to sign in with Microsoft:

1. Click **"Sign-in options"**
2. Click **"Domain join instead"**
3. Enter the following:

| Field | Value |
|---|---|
| Name | `labuser` |
| Password | Create a strong password |
| Security questions | Complete all three |

![Image](/images/lab-setup/windows-11/40-installation-36.png)
![Image](/images/lab-setup/windows-11/41-installation-37.png)
![Image](/images/lab-setup/windows-11/42-installation-38.png)
![Image](/images/lab-setup/windows-11/43-installation-39.png)
![Image](/images/lab-setup/windows-11/44-installation-40.png)

> **Important:** Remember these credentials - they are required every time
> we login to the VM.

---

### Step 8 - Privacy Settings

Turn **Off** all optional privacy settings:

| Setting | Value |
|---|---|
| Location | Off |
| Find my device | Off |
| Diagnostic data | Send required only |
| Inking and typing | Off |
| Tailored experiences | Off |
| Advertising ID | Off |

Click **Accept**.

---

### Step 9 - Skip Windows Updates

If prompted to download updates, click **"Update later"**.

![Image](/images/lab-setup/windows-11/46-installation-42.png)

> Windows 11 Enterprise Evaluation is now installed.
> A watermark appears in the bottom right corner showing the evaluation expiry
> date. This is normal and expected. The evaluation period is 90 days from
> installation.

![Image](/images/lab-setup/windows-11/47-installation-43.png)

---

## Part 5 - Install VMware Tools

VMware Tools must be installed to provide:

| Feature | Benefit |
|---|---|
| Display resolution | Proper scaling and resizing |
| Clipboard sharing | Copy and paste between host and VM |
| Drag and drop | File transfer between host and VM |
| Performance | Significantly improved VM responsiveness |
| Mouse integration | Seamless mouse movement in and out of VM |

---

### Step 1 - Launch VMware Tools Installer

With the VM powered on, right-click the VM name in the Library panel and click **"Install VMware Tools"**.

Inside the VM:

1. Open **File Explorer**
2. Click the **VMware Tools DVD drive**
3. Double-click **setup.exe**
4. Click **Yes** on the UAC prompt

![Image](/images/lab-setup/windows-11/48-installation-44.png)

---

### Step 2 - Complete VMware Tools Setup

| Screen | Action |
|---|---|
| Welcome | Click Next |
| Setup Type | Select Typical > Next |
| Ready to Install | Click Install |
| Completed | Click Finish |


![Image](/images/lab-setup/windows-11/49-installation-45.png)
![Image](/images/lab-setup/windows-11/50-installation-46.png)
![Image](/images/lab-setup/windows-11/51-installation-47.png)
![Image](/images/lab-setup/windows-11/52-installation-48.png)

Click **Yes** when prompted to restart.

![Image](/images/lab-setup/windows-11/53-installation-49.png)

> After restart the VM display will automatically resize to fill the VMware
> window correctly - this confirms VMware Tools installed successfully.

---

## Part 6 - Post-Installation Configuration

Open **PowerShell as Administrator** and run the following commands.

---

### Step 1 - Configure Static IP Address

**Check existing configuration:**
```powershell
ipconfig /all
```

![Image](/images/lab-setup/windows-11/54-post-installation-01.png)

**Remove existing DHCP assigned IP:**
```powershell
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -Confirm:$false
```

![Image](/images/lab-setup/windows-11/55-post-installation-02.png)

**Set static IP:**
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.48.128 -PrefixLength 24 -DefaultGateway 192.168.48.2
```

![Image](/images/lab-setup/windows-11/56-post-installation-03.png)

**Set DNS:**
```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 8.8.8.8
```

![Image](/images/lab-setup/windows-11/57-post-installation-04.png)

---

### Step 2 - Set Hostname

**Rename computer:**
```powershell
Rename-Computer -NewName "WIN11-CLIENT01" -Force -Restart
```

After the VM restarts, verify changes:
```powershell
ipconfig /all
```

![Image](/images/lab-setup/windows-11/58-post-installation-05.png)

Test network connectivity:
```powershell
Test-NetConnection 8.8.8.8
```

![Image](/images/lab-setup/windows-11/59-post-installation-06.png)

---

### Step 3 - Enable Audit and Logging Policies

**Enable Process Creation Auditing:**
```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```

Records every process launched on the system into Windows Security Event Log as **Event ID 4688**. Without this there is no native Windows log of which programs executed, when, and by whom. Required to capture command-line arguments of suspicious processes.

![Image](/images/lab-setup/windows-11/60-post-installation-07.png)

---

**Enable PowerShell Script Block Logging:**
```powershell
$path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $path -Force
New-ItemProperty -Path $path -Name "EnableScriptBlockLogging" -Value 1 -PropertyType DWORD -Force
```

Records the actual content of every PowerShell script executed as **Event ID 4104** - including obfuscated scripts delivered in memory. Without this, PowerShell attacks are invisible beyond the process name.

![Image](/images/lab-setup/windows-11/61-post-installation-08.png)

---

**Enable PowerShell Module Logging:**
```powershell
$path2 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging"
New-Item -Path $path2 -Force
New-ItemProperty -Path $path2 -Name "EnableModuleLogging" -Value 1 -PropertyType DWORD -Force
```

Records every PowerShell command and pipeline execution as **Event ID 4103**, including modules loaded and parameters passed. Complements Script Block Logging by capturing individual cmdlet execution that script blocks alone may miss.

![Image](/images/lab-setup/windows-11/62-post-installation-09.png)

---

**Set PowerShell Execution Policy:**
```powershell
Set-ExecutionPolicy Bypass -Scope LocalMachine -Force
```

Allows all lab scripts to execute without restriction.

![Image](/images/lab-setup/windows-11/63-post-installation-10.png)

> **Warning:** This is a lab-only setting. Never apply this in a production
> environment.

---

**Create Tools Directory:**
```powershell
New-Item -ItemType Directory -Path "C:\Tools" -Force
```

Central location for all tools installed on this VM including Sysmon, Mimikatz, PsExec, and custom scripts.

![Image](/images/lab-setup/windows-11/64-post-installation-11.png)

---

## Part 7 - Take Initial Snapshot

Before taking the snapshot, lock the screen: ```Win + L```

> **Why lock before snapshot?**
> Restoring a snapshot with an active logged-in session allows immediate access without a password. Locking the screen ensures every snapshot restore requires password authentication - correct security practice even in a lab environment.

In VMware:

```VM menu > Snapshot > Take Snapshot```

![Image](/images/lab-setup/windows-11/65-post-installation-12.png)

Give a name and some description for this and click on **"Take Snapshot"**.

![Image](/images/lab-setup/windows-11/66-post-installation-13.png)

> This snapshot is the recovery point if anything goes wrong during subsequent tool installations. We can restore to this snapshot at any time to return to a clean baseline state.

---
