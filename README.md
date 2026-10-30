# 🛡️ Cybersecurity Home Lab: VirtualBox & Kali Linux Setup

## 📖 Project Overview
This repository documents the setup of a secure, isolated cybersecurity lab environment designed for penetration testing and ethical hacking practice. The lab uses Oracle VirtualBox to host Kali Linux as an attacking machine, isolated on a custom NAT Network. 

This project was completed as **Week 1 - Project Module 1** of my Cybersecurity Internship.

## 🎯 Objectives & Requirements
*   **Hypervisor:** Oracle VirtualBox
*   **Attacking Machine:** Kali Linux
*   **Network Subnet:** `10.0.0.0/24` (NAT Network)
*   **Static IP:** `10.0.0.2/24` (with full internet access)
*   **Integration:** Bidirectional clipboard, drag-and-drop, and shared `/downloads` folder
*   **Recovery:** Baseline VM Snapshot created

## 🏗️ Architecture & Configuration
*   **Host OS:** Windows 10/11
*   **Virtual Network:** Custom NAT Network (`10.0.0.0/24`)
*   **Gateway:** `10.0.0.1` (VirtualBox NAT Engine)
*   **Kali IP:** `10.0.0.2/24`
*   **DNS:** `8.8.8.8` and `1.1.1.1`

## 🛠️ Key Technical Steps

### 1. VirtualBox NAT Network Creation
Created a dedicated NAT Network in VirtualBox with the IPv4 prefix `10.0.0.0/24` to isolate the lab traffic from the host machine.

### 2. Kali Linux VM Configuration
Attached Kali Linux to the new NAT Network. Configured shared folders to mount the host's `Downloads` directory and enabled bidirectional clipboard/drag-and-drop for seamless file transfer.

### 3. Static IP Assignment (CLI)
Configured a persistent static IP using `nmcli` inside the Kali terminal:
```bash
sudo nmcli connection add type ethernet con-name "Kali-Lab" ifname eth0 \
ipv4.method manual \
ipv4.addresses 10.0.0.2/24 \
ipv4.gateway 10.0.0.1 \
ipv4.dns "8.8.8.8 1.1.1.1"

# Critical step to bypass VirtualBox NAT address reservation
sudo nmcli connection modify "Kali-Lab" ipv4.dad-timeout 0

sudo nmcli connection up "Kali-Lab"
