# Technical Build Notes: Segmented Security Homelab

## Project Overview
This document outlines the technical configuration and deployment steps for the Segmented Security Homelab. The environment is designed to simulate an enterprise-grade network with isolated zones for production services and security research, now featuring a centralized Security Operations Center (SOC) capability.

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
* **SIEM Provisioning: Added a specific 'Pass' rule for the SIEM Manager (10.0.1.104) to allow outbound HTTPS/DNS traffic for installation and updates.

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
1.  **DNS Realignment:** Client network adapters (Win10, Ubuntu10) were manually configured to point to the Server 2022 IP as the Primary DNS resolver.
2. Ubuntu10 Integration: Replaced legacy Fedora instance with an Ubuntu Client to standardize security hardening documentation and agent compatibility.
3.  **Authentication:** Successful domain joins performed via the realm join (Linux) and System Properties (Windows) workflows.

## Section 4: SIEM & Security Monitoring (SOC)
Implemented centralized monitoring to provide deep-level visibility into host and network activity.

### Wazuh Manager Deployment
* OS: Debian 12 (Minimal/Headless).
* Specs: 8GB RAM, 4 vCPUs, 50GB Disk.
* Troubleshooting Note: Installation required manual DNS intervention. Modified /etc/resolv.conf to include nameserver 1.1.1.1 to bypass initial resolver conflicts.
* Credential Recovery: Managed installation credentials via a compressed .tar file. Used tar -O -xvf wazuh-install-files.tar to retrieve the admin password without exposing files to the clear-text directory.

### Endpoint Telemetry
* Windows Agents: Deployed via PowerShell (MSI) with specific flags for manager IP and agent naming conventions.
* Verification: Validated real-time log ingestion including failed logon attempts and File Integrity Monitoring (FIM) events.

## Section 5: Local AI & Docker Infrastructure (Host Level)
Research environment running on the physical host OS, logically separated from the virtualized lab.

* Infrastructure: Docker Engine running Ollama (LLMs) and ComfyUI.
* Future Scope: Integration of Docker container logs into the Wazuh SIEM for unified hybrid-cloud monitoring simulation.
