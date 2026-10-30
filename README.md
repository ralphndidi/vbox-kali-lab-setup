# Cybersecurity Testing Lab Setup: Kali Linux on VirtualBox

![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![OS](https://img.shields.io/badge/Kali-Linux-557C94)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)


## 📌 Project Overview
An isolated virtual penetration-testing lab built with Oracle VirtualBox
and Kali Linux on a custom NAT Network with a static IP.

**Objective:** Design, deploy, and configure a secure, isolated virtual testing lab environment using Oracle VirtualBox and Kali Linux. The lab is designed to simulate a real-world penetration testing environment.

## 🏗️ Infrastructure & Network Architecture
To prevent interference with the host machine's local network and to isolate the lab, a custom NAT Network was chosen over a standard Bridged Adapter.

| Component | Configuration |
| :--- | :--- |
| **Hypervisor** | Oracle VirtualBox |
| **Attacker Machine** | Kali Linux |
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


📸 Appendices (Project Screenshots)
1. VirtualBox NAT Network Configuration
https://images/01-nat-network-config.png

2. Kali VM Network Settings
https://images/02-kali-vm-network-adapter.png

3. NetworkManager Connection Profile
https://images/03-nmcli-static-config.png

4. Static IP Verification (ip a)
https://images/04-ip-a-eth0.png

5. Internet Connectivity Verification (ping 8.8.8.8)
https://images/05-internet-connectivity.png

6. Shared Folders Configuration
https://images/06-shared-folder-settings.png

7. Terminal Verification of Shared Folder (/media/sf_KaliShare)
https://images/07-shared-folder-in-kali.png

8. Baseline Snapshot Created
https://images/08-baseline-snapshot.png
