# Ubuntu Server LTS - Download & Installation Guide

## Overview

This guide covers the complete download, installation, and configuration of
Ubuntu Server 22.04.5 LTS inside a VMware Workstation Pro 17.

Ubuntu Server acts as the central server in the Purple Team home lab, hosting the following tools:

| Tool | Purpose |
|---|---|
| Wazuh | SIEM - receives and analyzes logs |
| MITRE Caldera | Adversary emulation platform |
| ATT&CK Navigator | Coverage analysis and visualization |
| Velociraptor | Advanced endpoint telemetry collection |

---

## Part 1 - Download Ubuntu Server LTS ISO

### Step 1 - Navigate to Ubuntu Official Download Page

Browse to the official Ubuntu Server [download](https://ubuntu.com/download/server) page.

![Image](/images/lab-setup/ubuntu-server/01-download.png)

Currently, latest version is `26.04`.

---

### Step 2 - Select Ubuntu Server Version

Sroll down to `Previous releases` section and click on **"Download 22.04.5 LTS"**.

![Image](/images/lab-setup/ubuntu-server/02-download.png)

Download should start now.

![Image](/images/lab-setup/ubuntu-server/03-download.png)

---

## Part 2 - Create a New Virtual Machine in VMware Workstation Pro 17

### Step 1 - Open New Virtual Machine Wizard

Go to `File > New Virtual Machine`.

![Image](/images/lab-setup/ubuntu-server/04-create-vm.png)

Select **"Custom (advanced)"** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/05-create-vm.png)

---

### Step 2 - Hardware Compatibility

| Setting | Value |
|---|---|
| Hardware compatibility | Workstation 17.x |

Click **Next**.

![Image](/images/lab-setup/ubuntu-server/06-create-vm.png)

---

### Step 3 - Guest OS Installation

Select **"I will install the operating system later"** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/07-create-vm.png)

---

### Step 4 - Select Guest Operating System

| Setting | Value |
|---|---|
| Guest operating system | Linux |
| Version | Ubuntu 64-bit |

Click **Next**.

![Image](/images/lab-setup/ubuntu-server/08-create-vm.png)

---

### Step 5 - Name and Location

| Setting | Value |
|---|---|
| Virtual machine name | `UBUNTU-SERVER01` |
| Location | `C:\VirtualMachines\UBUNTU-SERVER01` |

Click **Next**.

![Image](/images/lab-setup/ubuntu-server/09-create-vm.png)

---

### Step 6 - Processor Configuration

| Setting | Value |
|---|---|
| Number of processors | 1 |
| Number of cores per processor | 2 |

Click **Next**.

![Image](/images/lab-setup/ubuntu-server/10-create-vm.png)

> **Why 2 cores?**
> Wazuh, Caldera, and ATT&CK Navigator running simultaneously benefit from 2 cores. A single core will cause noticeable performance degradation when multiple services are running.

---

### Step 7 - Memory

Assign **4096 MB** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/11-create-vm.png)

> **Memory allocation for 16 GB host:**
>
> | VM | RAM |
> |---|---|
> | UBUNTU-SERVER01 | 4 GB |
> | WIN11-CLIENT01 | 4 GB |
> | Host OS | ~4 GB |
> | Buffer | ~4 GB |
> | **Total** | **16 GB** |

---

### Step 8 - Network Type

Select **NAT** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/12-create-vm.png)

---

### Step 9 - I/O Controller Type

Select **LSI Logic (Recommended)** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/13-create-vm.png)

---

### Step 10 - Disk Type

Select **SCSI (Recommended)** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/14-create-vm.png)

---

### Step 11 - Select a Disk

Select **"Create a new virtual disk"** and click **Next**.

![Image](/images/lab-setup/ubuntu-server/15-create-vm.png)

---

### Step 12 - Disk Capacity

| Setting | Value |
|---|---|
| Maximum disk size | 80 GB |
| Allocate all disk space now | Unchecked |
| Disk storage | Store virtual disk as a single file |

Click **Next**.

![Image](/images/lab-setup/ubuntu-server/16-create-vm.png)

---

### Step 13 - Disk File Name

Leave the default filename as suggested by VMware and click **Next**.

![Image](/images/lab-setup/ubuntu-server/17-create-vm.png)

---

### Step 14 - Customize Hardware

Click **"Customize Hardware"** before finishing.

![Image](/images/lab-setup/ubuntu-server/18-create-vm.png)

**CD/DVD (SATA):**

| Setting | Value |
|---|---|
| Connect at power on | Enabled |
| Connection | Use ISO image file |
| ISO path | Browse to downloaded ISO file |

![Image](/images/lab-setup/ubuntu-server/19-create-vm.png)

**Remove Sound Card:**
Select **Sound Card** and click **Remove**

![Image](/images/lab-setup/ubuntu-server/20-create-vm.png)

> Sound cards serve no purpose in a server VM and consume unnecessary resources.

**Display:**

| Setting | Value |
|---|---|
| Accelerate 3D graphics | Disabled |

> Ubuntu Server has no graphical desktop - 3D acceleration and high graphics memory are unnecessary.

Click **Close** 

![Image](/images/lab-setup/ubuntu-server/21-create-vm.png)

Click **Finish**.

![Image](/images/lab-setup/ubuntu-server/22-create-vm.png)

---

## Part 3 - Install Ubuntu Server

### Step 1 - Power On the VM

Click **"Power on this virtual machine"**.

![Image](/images/lab-setup/ubuntu-server/23-install-vm.png)

---

### Step 2 - GNU GRUB Boot Menu

When the GRUB menu appears, select ```Try or Install Ubuntu Server``` and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/24-install-vm.png)

---

### Step 3 - Select Language

Select **English** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/25-install-vm.png)

---

### Step 4 - Installer Update Prompt

If the installer offers an update to itself, select **"Continue without updating"**

![Image](/images/lab-setup/ubuntu-server/26-install-vm.png)

> Updating the installer requires Internet access and adds unnecessary time at this stage. System updates will be performed after installation.

---

### Step 5 - Keyboard Configuration

| Setting | Value |
|---|---|
| Layout | English (US) |
| Variant | English (US) |

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/27-install-vm.png)

---

### Step 6 - Installation Type

Select **"Ubuntu Server"** (not minimized), select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/28-install-vm.png)

> **Why not minimized?**
>
> The standard Ubuntu Server installation includes essential networking tools, utilities, and package management components needed for lab tool installation. The minimized version removes too many components that will need to be reinstalled manually.

---

### Step 7 - Network Configuration

The installer will detect the network interface and attempt DHCP.

Note the IP address assigned — it will look similar to: ```DHCPv4: 192.168.x.x/24```

![Image](/images/lab-setup/ubuntu-server/29-install-vm.png)

Leave as DHCP for now. Static IP will be configured after installation.

Select **Done** and press **Enter**.

---

### Step 8 - Proxy Configuration

Leave proxy blank unless your network requires one.

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/30-install-vm.png)

---

### Step 9 - Ubuntu Archive Mirror

Leave the default mirror: `http://in.archive.ubuntu.com/ubuntu`

> Note: The installer tests the mirror. Wait for the connectivity check to complete before proceeding.

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/31-install-vm.png)

---

### Step 10 - Storage Configuration

Select **"Use an entire disk"** and press **Enter**.

| Setting | Value |
|---|---|
| Storage configuration | Use an entire disk |
| Set up this disk as an LVM group | Enabled (default) |

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/32-install-vm.png)

Review the storage summary, select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/33-install-vm.png)

When prompted **"Confirm destructive action"**, select **Continue** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/34-install-vm.png)

---

### Step 11 - Profile Configuration

Configure the server identity and credentials.

| Field | Recommended Value |
|---|---|
| Your name | `Lab Admin` |
| Your server's name | `ubuntu-server01` |
| Pick a username | `labadmin` |
| Password | Create a strong password |
| Confirm password | Re-enter password |

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/35-install-vm.png)

---

### Step 12 - Ubuntu Pro

Choose **"Skip for now"**, select **Continue** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/36-install-vm.png)

> Ubuntu Pro is a commercial subscription. It is not required for a home lab.

---

### Step 13 - SSH Configuration

| Setting | Value |
|---|---|
| Install OpenSSH server | Enabled |

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/37-install-vm.png)

---

### Step 14 - Featured Server Snaps

Leave all snaps **unselected**.

Select **Done** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/38-install-vm.png)

---

### Step 15 - Installation Progress

Ubuntu Server installation begins automatically.

![Image](/images/lab-setup/ubuntu-server/39-install-vm.png)

Wait for the ```Installation complete!``` message.

Select **"Reboot Now"** and press **Enter**.

![Image](/images/lab-setup/ubuntu-server/40-install-vm.png)

---

### Step 16 - Remove ISO After Reboot

When prompted - ```Please remove the installation medium, then press ENTER```

![Image](/images/lab-setup/ubuntu-server/41-install-vm.png)

In VMware: ```VM menu > Settings > CD/DVD (SATA) > Disconnect```

![Image](/images/lab-setup/ubuntu-server/42-install-vm.png)

Press **Enter** in the VM to complete the reboot.

---

### Step 17 - First Login

When the login prompt appears:

```
ubuntu-server01 login: labadmin
Password: [enter password]
```

![Image](/images/lab-setup/ubuntu-server/43-install-vm.png)

Successful login shows: `labadmin@ubuntu-server01:~$`

![Image](/images/lab-setup/ubuntu-server/44-install-vm.png)

---

## Part 4 - Post-Installation Configuration

### Step 1 - Configure Static IP Address

Ubuntu Server uses Netplan to manage network configuration.

**Step 1.1 - Identify Network Information:**

```bash
ip addr show && ip route
```

![Image](/images/lab-setup/ubuntu-server/45-post-installation.png)

Note the below information from above output:
```
Interface     = ens33
Server IP     = 192.168.48.129
Subnet Mask   = /24
Gateway       = 192.168.48.2
Network       = 192.168.48.0/24
```

**Step 1.2 - Edit Netplan Configuration:**

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

![Image](/images/lab-setup/ubuntu-server/46-post-installation.png)

Replace the entire file content with:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.48.129/24
      routes:
        - to: default
          via: 192.168.48.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

![Image](/images/lab-setup/ubuntu-server/47-post-installation.png)

> **Important:**
> - Replace `ens33` with actual interface name from Step 1.1
> - Replace `192.168.48.129` with chosen static IP for this server
> - Replace `192.168.48.2` with VMware NAT gateway IP
> - YAML is indent-sensitive - use spaces, never tabs

Save the file: ```"Ctrl + O" > "Enter" > "Ctrl + X"```

Alternatively, use below heredoc command:

```bash
sudo tee /etc/netplan/00-installer-config.yaml > /dev/null << 'EOF'
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.48.129/24
      routes:
        - to: default
          via: 192.168.48.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
EOF
```

**Step 1.3 — Disable Cloud-Init Network Configuration (Recommended):**

Ubuntu Server installations may create an additional Netplan file
through Cloud-Init that enables DHCP. If left in place, the server
can receive a second IP address from DHCP in addition to the
configured static IP.

Check whether the Cloud-Init Netplan file exists and remove:

```bash
# Check if Cloud-Init Netplan file exists
ls -l /etc/netplan/

# Read the content of Cloud-Init Netplan file
sudo cat /etc/netplan/50-cloud-init.yaml

# Backup the Cloud-Init Netplan file
sudo cp /etc/netplan/50-cloud-init.yaml /etc/netplan/50-cloud-init.yaml.bak

# Remove the Cloud-Init Netplan file
sudo rm /etc/netplan/50-cloud-init.yaml
```

![Image](/images/lab-setup/ubuntu-server/47-post-installation-01.png)

Prevent Cloud-Init from recreating DHCP network settings in future boots:
```bash
sudo mkdir -p /etc/cloud/cloud.cfg.d
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

![Image](/images/lab-setup/ubuntu-server/47-post-installation-02.png)

Add the following line:
```
network: {config: disabled}
```

![Image](/images/lab-setup/ubuntu-server/47-post-installation-03.png)

Save and exit: `"Ctrl + O" > "Enter" > "Ctrl + X"`

**Step 1.4 - Apply Netplan Configuration:**

Set appropriate permission for the conf file:

```bash
sudo chmod 600 /etc/netplan/00-installer-config.yaml
```

Generate and apply the new network configuration:

```bash
sudo netplan generate
sudo netplan apply
```

Restart the networking service:
```bash
sudo systemctl restart systemd-networkd
```

![Image](/images/lab-setup/ubuntu-server/48-post-installation.png)

> The warning ```WARNING:root:Cannot call Open vSwitch: ovsdb-server.service is not running.``` means:
>
> Netplan checked whether Open vSwitch (OVS) is available, but the OVS service is not running.
>
> It does not mean network configuration failed and warning can be safely ignored.

**Step 1.5 - Verify Static IP:**

```bash
ip addr show ens33
```

![Image](/images/lab-setup/ubuntu-server/49-post-installation.png)

---

### Step 2 - Verify Network Connectivity

**Test Internet Connectivity:**

```bash
ping -c 4 8.8.8.8
```

![Image](/images/lab-setup/ubuntu-server/50-post-installation.png)

Expected output: ```4 packets transmitted, 4 received, 0% packet loss```

**Test DNS Resolution:**

```bash
ping -c 4 google.com
```

![Image](/images/lab-setup/ubuntu-server/51-post-installation.png)

---

### Step 3 - System Updates

```bash
# Update package list
sudo apt-get update

# Upgrade all packages
sudo apt-get upgrade -y

# Upgrade distribution packages
sudo apt-get dist-upgrade -y

# Remove unnecessary packages
sudo apt-get autoremove -y

# Clean package cache
sudo apt-get clean
```

While upgrading all packages, when prompted for ```Which services should be restarted?```, proceed with default selection and use ```TAB``` to go to **"OK"** and hit **ENTER**.

![Image](/images/lab-setup/ubuntu-server/52-post-installation.png)

**Verify system is up to date:**

```bash
sudo apt-get update && sudo apt-get upgrade --dry-run
```

![Image](/images/lab-setup/ubuntu-server/53-post-installation.png)

Expected output ends with: ```0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.```

---

### Step 4 - Configure SSH for Remote Administration

SSH was enabled during installation. Harden the SSH configuration for lab use.

**Edit SSH configuration:**

```bash
sudo nano /etc/ssh/sshd_config
```

Locate and update the following settings:

```bash
# SSH port
Port 22

# Disable root login
PermitRootLogin no

# Enable password authentication (required for lab access)
PasswordAuthentication yes

# Limit login attempts
MaxAuthTries 5

# Set login grace time
LoginGraceTime 30
```

![Image](/images/lab-setup/ubuntu-server/54-post-installation.png)
![Image](/images/lab-setup/ubuntu-server/55-post-installation.png)

Save: ```"Ctrl + O" > "Enter" > "Ctrl + X"```

**Restart SSH service:**

```bash
sudo systemctl restart ssh
```

**Verify SSH is running:**

```bash
sudo systemctl status ssh
```

![Image](/images/lab-setup/ubuntu-server/56-post-installation.png)

**Test SSH from Windows 11 host:**

Open PowerShell on Windows 11 host machine and run:

```powershell
ssh labadmin@192.168.48.129
```

![Image](/images/lab-setup/ubuntu-server/57-post-installation.png)

Successful connection shows: ```labadmin@ubuntu-server01:~$```

> **From this point forward**, all Ubuntu Server administration can be performed via SSH from Windows 11 host.

---

### Step 5 - Configure UFW Firewall

**Enable UFW:**

```bash
sudo ufw enable
```

![Image](/images/lab-setup/ubuntu-server/58-post-installation.png)

**Set default policies:**

```bash
# Deny all incoming by default
sudo ufw default deny incoming

# Allow all outgoing by default
sudo ufw default allow outgoing
```

![Image](/images/lab-setup/ubuntu-server/59-post-installation.png)

**Allow required ports:**

```bash
# SSH - remote administration
sudo ufw allow 22/tcp

# Wazuh agent communication
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp

# Wazuh API
sudo ufw allow 55000/tcp

# Wazuh dashboard (HTTPS)
sudo ufw allow 443/tcp

# MITRE Caldera
sudo ufw allow 8888/tcp

# ATT&CK Navigator
sudo ufw allow 4200/tcp

# Velociraptor
sudo ufw allow 8000/tcp
```

![Image](/images/lab-setup/ubuntu-server/60-post-installation.png)

**Verify firewall rules:**

```bash
sudo ufw status verbose
```

![Image](/images/lab-setup/ubuntu-server/61-post-installation.png)

---

### Step 6 - Basic Linux Hardening

Apply these hardening steps appropriate for a home lab environment:

**Step 6.1 - Disable Unnecessary Services:**

```bash
# Check running services
sudo systemctl list-units --type=service --state=running

# Disable services not needed in lab (if present)
sudo systemctl disable --now snapd
sudo systemctl disable --now bluetooth
sudo systemctl disable --now cups
```

![Image](/images/lab-setup/ubuntu-server/62-post-installation.png)
![Image](/images/lab-setup/ubuntu-server/63-post-installation.png)

**Step 6.2 - Configure Automatic Security Updates:**

```bash
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Select **Yes** when prompted.

![Image](/images/lab-setup/ubuntu-server/64-post-installation.png)

**Step 6.3 - Set Login Banner:**

```bash
sudo nano /etc/issue.net
```

Add:

```
*************************************************************
*  PURPLE TEAM HOME LAB — AUTHORIZED ACCESS ONLY           *
*  Unauthorized access is strictly prohibited              *
*************************************************************
```

Save: `"Ctrl + O" > "Enter" > "Ctrl + X"`

![Image](/images/lab-setup/ubuntu-server/65-post-installation.png)

**Step 6.4 - Enable the banner in SSH:**

```bash
sudo nano /etc/ssh/sshd_config
```

Uncomment and set:

```Banner /etc/issue.net```

![Image](/images/lab-setup/ubuntu-server/66-post-installation.png)

Restart SSH:

```bash
sudo systemctl restart ssh
```

**Step 6.5 - Set Timezone:**

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

![Image](/images/lab-setup/ubuntu-server/67-post-installation.png)

> Replace `Asia/Kolkata` with applicable timezone.
> List available timezones: `timedatectl list-timezones`

**Step 6.6 - Verify timezone:**

```bash
timedatectl status
```

![Image](/images/lab-setup/ubuntu-server/68-post-installation.png)

**Step 6.7 - Install Essential Tools:**

| Tool | Package Name | Binary Name | Purpose |
|---|---|---|---|
| curl | `curl` | `curl` | HTTP requests and file downloads |
| wget | `wget` | `wget` | File downloads from command line |
| git | `git` | `git` | Version control for lab configurations |
| net-tools | `net-tools` | `ifconfig` / `netstat` | Network interface and connection inspection |
| htop | `htop` | `htop` | Interactive process and resource monitor |
| tree | `tree` | `tree` | Visual directory structure display |
| unzip | `unzip` | `unzip` | Extract ZIP archives |
| Python 3 | `python3` | `python3` | Runtime for Caldera and lab automation scripts |
| pip | `python3-pip` | `pip3` | Python package installer |
| Docker | `docker.io` | `docker` | Container platform for Wazuh deployment |
| Docker Compose | `docker-compose` | `docker-compose` | Multi-container orchestration for Wazuh stack |

Check what essential tool binaries are already installed:

```bash
command -v curl wget git ifconfig htop tree unzip python3 pip3 docker docker-compose
```

![Image](/images/lab-setup/ubuntu-server/69-post-installation.png)

Install the missing tool packages:

```bash
sudo apt-get install -y net-tools tree unzip python3-pip docker.io docker-compose
```

![Image](/images/lab-setup/ubuntu-server/70-post-installation.png)

Verify all tools have been successfully installed:

```bash
command -v curl wget git ifconfig htop tree unzip python3 pip3 docker docker-compose
```

![Image](/images/lab-setup/ubuntu-server/71-post-installation.png)

**Enable Docker:**

```bash
sudo systemctl enable docker
sudo systemctl start docker

# Add labadmin to docker group (avoids sudo for every docker command)
sudo usermod -aG docker labadmin

# Log out and back in for group change to take effect
exit
```

![Image](/images/lab-setup/ubuntu-server/72-post-installation.png)

Log back in via SSH:

```bash
ssh labadmin@192.168.48.128
```

**Verify Docker:**

```bash
docker --version
```

![Image](/images/lab-setup/ubuntu-server/73-post-installation.png)

---

### Step 7 - Verify Memory & Disk Space

**Memory:**

```bash
free -h
```

> Confirm 4 GB total memory visible.

![Image](/images/lab-setup/ubuntu-server/74-post-installation.png)

**Disk space:**

```bash
df -h /
```

> Confirm ~80 GB available on root partition.

![Image](/images/lab-setup/ubuntu-server/75-post-installation.png)

We only have around 40 GB assigned, not 80 GB. Ubuntu Server 22.04 LTS installer uses LVM (Logical Volume Manager) by default and intentionally allocates only half the disk space to the root logical volume, leaving the rest unallocated as free space for future use. This is a known Ubuntu Server installer behavior.

Verify unallocated space exists:

```bash
sudo vgdisplay | grep -E "VG Size|Alloc|Free"
```

![Image](/images/lab-setup/ubuntu-server/76-post-installation.png)

> Confirm ~40 GB is sitting unallocated inside the volume group.

Extend the logical volume to 100% of available space:

```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Resize the filesystem to use the new space:

```bash
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Verify full disk is now available:

```bash
df -h /
```

![Image](/images/lab-setup/ubuntu-server/77-post-installation.png)

---

## Part 5 - Take Snapshot

### Step 1 - Clean Up Before Snapshot

```bash
# Clear bash history
history -c && history -w

# Exit SSH session
exit
```

![Image](/images/lab-setup/ubuntu-server/78-post-installation.png)

> `history -c`: Clears the in-memory bash history for the current session
>
> `history -w`: Writes the cleared history to the .bash_history file on disk

### Step 2 - Take Snapshot

In VMware, with the VM running:

```
VM menu > Snapshot > Take Snapshot
```

| Field | Value |
|---|---|
| Name | `Clean Install` |
| Description | `Static IP configured. SSH enabled. UFW configured. Docker installed. No security tools deployed yet.` |

Click **"Take Snapshot"**.

![Image](/images/lab-setup/ubuntu-server/79-post-installation.png)

---
