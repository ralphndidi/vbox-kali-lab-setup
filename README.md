# Cybersecurity Testing Lab Setup: Kali Linux on VirtualBox

![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![OS](https://img.shields.io/badge/Kali-Linux-557C94)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)


## 📌 Project Overview
An isolated virtual penetration-testing lab built with Oracle VirtualBox
and Kali Linux on a custom NAT Network with a static IP.

**Objective:** Design, deploy, and configure a secure, isolated virtual testing lab environment using Oracle VirtualBox and Kali Linux. The lab is designed to simulate a real-world penetration testing environment.

## 🏗️ Infrastructure & Network Architecture# Cybersecurity Testing Lab Setup: Kali Linux on VirtualBox

![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![OS](https://img.shields.io/badge/Kali-Linux-557C94)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

An isolated virtual penetration-testing lab built with Oracle VirtualBox
and Kali Linux on a custom NAT Network with a static IP.

## Objectives
- Build an isolated lab network separate from existing VMs
- Deploy Kali Linux with a static IP (10.0.0.2/24)
- Enable internet access, shared folders, clipboard and drag-and-drop
- Create a baseline snapshot for recovery

## Architecture
| Component | Value |
|---|---|
| Hypervisor | Oracle VirtualBox 7.x (Host: Windows 10) |
| Guest OS | Kali Linux <version> |
| Network | NAT Network `NatNetwork` |
| Subnet | 10.0.0.0/24 |
| Gateway | 10.0.0.1 |
| Kali IP | 10.0.0.2/24 (static) |

Existing VMs (Metasploitable2, etc.) remain on the separate
`LABNETWORK` (10.0.2.0/24) and were not affected.

## Setup Steps
1. **Create the NAT Network** (File → Tools → Network Manager)
   ![NAT](01-nat-network-config.png)
2. **Attach Kali's Adapter 1** to the NAT Network
   ![Adapter](02-kali-vm-network-adapter.png)
3. **Configure the static IP** 
4. **Set up the shared folder** with Auto-mount
5. **Take a baseline snapshot**

## Verification
| Test | Command | Result |
|---|---|---|
| Interface | `ip a` | eth0 UP, 10.0.0.2/24 |
| Gateway | `ping -c 3 10.0.0.1` | 0% loss |
| Internet | `ping -c 3 8.8.8.8` | 0% loss |
| DNS | `ping -c 3 google.com` | Resolves |

(embed screenshots 03–08 here)

## Troubleshooting: NetworkManager IP Reservation Timeout
**Problem:** `nmcli` failed with "IP configuration could not be reserved",
yet `ip addr add` worked.
**Likely cause:** Duplicate Address Detection conflicting with
VirtualBox's NAT engine reservations.
**Fix:** Disable DAD on the profile: `nmcli connection modify "Kali-Lab" ipv4.dad-timeout 0`

## Lessons Learned
- How VirtualBox NAT Networks differ from NAT and Bridged modes
- How NetworkManager handles static addressing and DAD
- Why snapshots matter before security testing

## Disclaimer
For educational use in an isolated lab only. Only test systems you own
or have written permission to test.

## Author
Raphael Ndidi-Uboma · [linkedin.com/in/ralphndidi](#) · Network Walks 2026 Cybersecurity Internship, Week 1

To prevent interference with the host machine's local network and to isolate the lab, a custom NAT Network was chosen over a standard Bridged Adapter.

| Component | Configuration |
| :--- | :--- |
| **Hypervisor** | Oracle VirtualBox |
| **Attacker Machine** | Kali Linux (version 2026.3) |
| **Network Type** | NAT Network (Custom) |
| **Subnet** | `10.0.0.0/24` |
| **Gateway** | `10.0.0.1` (VirtualBox NAT Engine) |
| **Kali Linux IP** | `10.0.0.2/24` (Static) |

> *Note: Pre-existing lab VMs (e.g., Metasploitable, Windows 11) residing on a separate `10.0.2.0/24` NAT Network were kept intact and unaffected to preserve the existing lab environment.*

---

## ⚙️ Implementation Steps

### Phase 1: VirtualBox & Network Setup
1.  **NAT Network Creation:** Created a new NAT network named `NatNetwork` with the IPv4 Prefix set to `10.0.0.0/24` and DHCP enabled.
2.  **Kali Import:** Imported the official Kali Linux VirtualBox image into the hypervisor.
3.  **Adapter Configuration:** Attached Adapter 1 to the custom NAT Network, ensuring the "Virtual Cable Connected" option was checked.

### Phase 2: Virtual Machine Integration
1.  **Integration Features:** Enabled bidirectional Clipboard and Drag-and-Drop.
2.  **Shared Folders:** Configured a permanent shared folder mapping the host's directory to the Kali VM, with Auto-mount enabled.

### Phase 3: IP Configuration & Troubleshooting
1.  Attempted to configure the IP via NetworkManager (`nmcli`) using the `manual` method.
2.  Encountered a virtualization conflict resulting in an IP reservation timeout. 
3.  Bypassed Duplicate Address Detection (DAD) to successfully assign the static IP `10.0.0.2/24`.
4.  Verified connectivity to the gateway (`10.0.0.1`) and external internet (`8.8.8.8`).

### Phase 4: Finalization
1.  Took a baseline snapshot of the fully configured Kali VM named "Week 1 - Kali Lab Setup Complete" to ensure a rollback point is available.

---

## 🧪 Verification & Testing

To ensure the environment met all requirements, the following tests were performed inside the Kali Linux terminal:

*   **Interface Check:** Ran `ip a`. Verified `eth0` had the correct static IP `10.0.0.2/24` and was in an `UP` state.
*   **Internet Ping:** Ran `ping -c 4 8.8.8.8`. Received 0% packet loss, confirming full external internet routing through the NAT engine.
*   **Shared Folder Access:** Ran `ls -la /media/sf_KaliShare` to verify the host machine's directory was successfully mounted inside the guest OS.

---

## 🛠️ Technical Challenge & Resolution (Root Cause Analysis)

**The Issue:**
During static IP assignment (`10.0.0.2/24`), NetworkManager (`nmcli`) repeatedly failed to activate the connection, returning the error: 
`Error: Connection activation failed: IP configuration could not be reserved (no available address, timeout, etc.)`
However, manually assigning the IP using the `ip addr add` command worked perfectly.

**The Diagnosis:**
This indicated a conflict between Kali's NetworkManager and VirtualBox's internal NAT engine. When NetworkManager activates a manual IP, it performs a Duplicate Address Detection (DAD) check. VirtualBox's internal NAT engine reserves the earliest addresses in the subnet (including `.2`) for its own DNS/DHCP proxy services. When NetworkManager asked "is `.2` in use?", VirtualBox responded, causing NetworkManager to abort the connection.

**The Resolution:**
To force NetworkManager to ignore the virtual reservation, a new connection profile (`Kali-Lab`) was created via the CLI, and the DAD timeout was disabled:

```bash
sudo nmcli connection add type ethernet con-name "Kali-Lab" ifname eth0 ipv4.method manual ipv4.addresses 10.0.0.2/24 ipv4.gateway 10.0.0.1 ipv4.dns "8.8.8.8 1.1.1.1"
sudo nmcli connection modify "Kali-Lab" ipv4.dad-timeout 0
sudo nmcli connection up "Kali-Lab"
**
