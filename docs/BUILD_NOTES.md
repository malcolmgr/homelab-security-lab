# Technical Build Notes: Segmented Security Homelab

## Project Overview
This document outlines the technical configuration and deployment steps for the Segmented Security Homelab. The environment is designed to simulate an enterprise-grade network with isolated zones for production services and security research.

## Section 1: pfSense Virtual Gateway & Networking
The pfSense firewall acts as the primary security gateway, utilizing a "Router-on-a-Stick" configuration via VMware virtual switches (VMnets).

### Physical & Virtual Interface Mapping
* **Host Hardware:** Custom Windows-based workstation (All-Black Build).
* **Network Interface Card (NIC):** **Intel i350-T2 (Dual-Port Gigabit Server Adapter)**. 
    * *Note:* Selected for its native driver support in FreeBSD/pfSense and hardware-level virtualization features.
* **WAN (em0):** Bridged to the first port of the Intel i350; interfaces with the **TP-Link Deco BE63 (Wi-Fi 7)** gateway.
* **VNet1 - Defensive (em1):** Static IP: `10.0.1.1/24`. Assigned to VMnet2.
* **VNet2 - Offensive (em2):** Static IP: `10.0.2.1/24`. Assigned to VMnet3.

### Firewall Policy (Network Segmentation)
* **Isolation Rule:** Implemented a 'Block' rule on the VNet2 interface to prevent any traffic from initiating a connection to the VNet1 (10.0.1.0/24) subnet.
* **Stateful Inspection:** Configured pfSense to log all dropped packets at the boundary to monitor for unauthorized lateral movement attempts.

---

## Section 2: Active Directory & Identity Management
The defensive segment (VNet1) is managed by a centralized Identity and Access Management (IAM) suite.

### Windows Server 2022 Deployment
* **Role:** Domain Controller (DC) promoted to a new forest.
* **DNS/DHCP Migration:** To maintain a "Standard Environment," DHCP services were disabled on the pfSense VNet1 interface. The Windows Server was configured as the authoritative DHCP and DNS provider for the `10.0.1.0/24` segment.
* **Hardening:** Basic GPO (Group Policy Object) implementation for password complexity and restricted login attempts.

---

## Section 3: Client Integration & Validation
Simulating a multi-OS corporate environment with integrated security monitoring.

### Domain Join Workflow
1.  **DNS Realignment:** Client network adapters (Win10, Ubuntu) were manually configured to point to the Server 2022 IP as the Primary DNS resolver.
2.  **Authentication:** Successful domain joins were performed using delegated admin credentials.
3.  **Fedora Web Server:** Configured a Fedora Headless instance within VNet1 to act as a localized web resource for cross-zone traffic testing.

### Connectivity Verification
* **Layer 3 Validation:** Performed bidirectional `ping` and `traceroute` tests to verify routing tables.
* **DNS Resolution:** Verified via `nslookup` that clients correctly resolve the Domain Controller and internal Fedora web services.