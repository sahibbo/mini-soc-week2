# Mini-SOC Network Topology

## Wazuh Mini-SOC — Week 1

```mermaid
flowchart TB
    INTERNET["Internet"]
    NAT["Management / NAT Network<br/>192.168.22.0/24<br/>Gateway: 192.168.22.2"]

    subgraph SOC["Isolated SOC-LAN — 10.10.10.0/24"]
        SIEM["siem-host<br/>Ubuntu Server 24.04<br/>10.10.10.10<br/>Wazuh Manager / Indexer / Dashboard"]
        WIN["win-client<br/>Windows 10<br/>10.10.10.20<br/>Wazuh Agent + Sysmon"]
        WEB["ubuntu-web<br/>Ubuntu Server 24.04<br/>10.10.10.30<br/>Wazuh Agent + Apache"]

        WIN <-->|"Wazuh Agent Traffic"| SIEM
        WEB <-->|"Wazuh Agent Traffic"| SIEM
    end

    INTERNET --> NAT
    NAT -->|"ens4: DHCP"| SIEM

    HOST["Analyst Workstation"]
    HOST -.->|"HTTPS Dashboard"| SIEM
    HOST -.->|"HTTP Apache"| WEB
```

## Network Security

- Only siem-host has a permanent NAT connection.
- win-client and ubuntu-web have no default IPv4 routes.
- Wazuh agents communicate through the internal SOC-LAN.
- The SIEM host uses a separate interface for management connectivity.
- The environment runs inside PNETLab on VMware Workstation.

## Related Documentation

See [Network Addressing](addressing.md) for detailed IP configurations and connectivity verification.
