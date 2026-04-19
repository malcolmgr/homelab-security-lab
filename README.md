# Segmented Security Homelab

## Architecture
- **Host**: VMware Workstation on Windows PC with dual-NIC (2-port add-on card)
- **Firewall**: pfSense VM acting as router-on-a-stick
- **VNet1 (Defensive/Corp)** : Windows Server 2022 (AD, DHCP, DNS), Windows 10 Client, Ubuntu 22.04 Client
- **VNet2 (Offensive/Isolated)** : Kali Linux, basic_pentesting_1 (TryHackME-style), OWASP Broken Web Apps v1.2

## Security Controls
- pfSense firewall rules block VNet2 from initiating traffic to VNet1
- VNet1 can log but not interact with VNet2
- All internet traffic passes through pfSense with logging enabled

## What I Practice Here
- **Blue Team**: AD hardening, log analysis, Windows/Linux defense
- **Red Team**: Web app pentesting (OWASP BWA), enumeration & privilege escalation (basic_pentesting_1)
- **Network Security**: pfSense rule writing, VLAN segmentation, traffic inspection

## Diagram
![Network Diagram](./network-diagram.png)