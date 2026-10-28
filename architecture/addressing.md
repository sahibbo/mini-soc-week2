# Mini-SOC Network Architecture and Addressing

## 1. Architecture Overview

The Week 1 Mini-SOC laboratory was deployed using VMware Workstation and PNETLab.

The environment contains three virtual machines and two logically separated networks.

The SIEM host provides centralized security monitoring using Wazuh 4.14.8.

## 2. Virtual Machine Inventory

| Hostname | Operating System | RAM | vCPU | Role |
|---|---|---|---|---|
| siem-host | Ubuntu Server 24.04.2 LTS | 16 GB | 6 | Wazuh SIEM |
| win-client | Windows 10 Enterprise Evaluation | 4 GB | 2 | Windows Endpoint |
| ubuntu-web | Ubuntu Server 24.04.2 LTS | 2 GB | 2 | Linux Web Server |

## 3. SOC-LAN Network

Network: 10.10.10.0/24

Purpose: Internal communication between the SIEM and monitored endpoints.

| Hostname | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| siem-host | 10.10.10.10 | 255.255.255.0 | Via separate NAT interface |
| win-client | 10.10.10.20 | 255.255.255.0 | None |
| ubuntu-web | 10.10.10.30 | 255.255.255.0 | None |

Both endpoints communicate with the SIEM host through the isolated SOC-LAN.

## 4. Management and NAT Network

Network: 192.168.22.0/24

The SIEM host uses a second network interface for management and external connectivity.

| Parameter | Configuration |
|---|---|
| Hostname | siem-host |
| Network Interface | ens4 |
| Address Assignment | DHCP |
| Network | 192.168.22.0/24 |
| Default Gateway | 192.168.22.2 |
| Purpose | Package downloads and management |

The management IP is DHCP-assigned and may change.

Only the SIEM host has a permanent NAT connection.

The Windows and Ubuntu endpoints do not have permanent default IPv4 routes.

## 5. SIEM Host Interfaces

| Interface | Network | Configuration |
|---|---|---|
| ens3 | SOC-LAN | Static: 10.10.10.10/24 |
| ens4 | Management/NAT | DHCP |

## 6. Network Services

| Service | Host | Port |
|---|---|---|
| Wazuh Dashboard | siem-host | TCP 443 |
| Wazuh Agent Communication | siem-host | TCP 1514 |
| Wazuh Agent Enrollment | siem-host | TCP 1515 |
| SSH | Ubuntu Servers | TCP 22 |
| Apache HTTP | ubuntu-web | TCP 80 |

Dashboard URL:
https://10.10.10.10

Apache URL:
http://10.10.10.30

## 7. Connectivity Verification

Bidirectional ICMP connectivity was successfully verified between all three virtual machines.

The tests returned 0% packet loss.

The Wazuh Dashboard was successfully accessed from the physical host browser.

The Apache default page was also accessible from the physical host browser.

## 8. Network Isolation Verification

The Ubuntu endpoint was checked using:

    ip -4 route

The Windows endpoint was checked using:

    Get-NetRoute -DestinationPrefix "0.0.0.0/0" -AddressFamily IPv4 -ErrorAction SilentlyContinue

Neither endpoint had a default IPv4 route.

This configuration limits external network access while preserving internal SOC communication.

## 9. Security Design

- SOC-LAN separates endpoint monitoring traffic from external connectivity.
- NAT connectivity is provided only to the SIEM host.
- Wazuh Agents communicate with the Manager using internal IP addresses.
- Endpoint internet access is not required for normal security monitoring.
- Installation packages can be transferred internally using SCP or a temporary HTTP server.
- The laboratory is intended for authorized security testing.
