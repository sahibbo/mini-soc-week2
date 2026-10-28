# Mini-SOC in a Box — Week 2

**Blue Team Capstone | LabShop Security Operations Center**

## Project Overview

Week 2 extends the Wazuh Mini-SOC environment deployed during Week 1 with advanced detection engineering, endpoint integrity monitoring, network intrusion detection, and security posture assessment.

**Previous phase:** [Mini-SOC Week 1](https://github.com/sahibbo/mini-soc-week1)

## Lab Environment

| Host | IP Address | Role |
|---|---|---|
| siem-host | 10.10.10.10 | Wazuh SIEM 4.14.8 |
| win-client | 10.10.10.20 | Windows Security Monitoring |
| ubuntu-web | 10.10.10.30 | Linux Web Server, Suricata IDS |

**Environment:** VMware Workstation / PNETLab
**Network:** Isolated SOC-LAN (10.10.10.0/24)

## Task 1 — Custom Detection Engineering

- Developed a LabShop authentication log generator.
- Implemented a custom Wazuh decoder using PCRE2.
- Rule 100100: Failed login detection (Level 5).
- Rule 100101: Brute-force correlation (Level 12).
- Correlation: 5 failures within 60 seconds from the same source IP.
- Excluded the monitoring account to reduce noisy alerts.
- MITRE ATT&CK: T1110 — Brute Force.

**Result:** Detection and correlation validated using Wazuh Logtest and live events.

## Task 2 — File Integrity Monitoring

- Linux real-time FIM: /var/www/html
- Linux scheduled FIM: /etc
- Windows real-time FIM: C:\LabShopData
- Windows Registry monitoring: HKLM\SOFTWARE\LabShop
- Validated file creation, modification, and deletion.
- Verified content differences through syscheck.diff.

**Result:** Linux and Windows FIM detection confirmed.

## Task 3 — Suricata IDS Integration

- Installed Suricata 7.0.3 on ubuntu-web.
- Monitored network interface ens3.
- Created HTTP /admin detection signature (SID 1000001).
- Forwarded Suricata EVE JSON events to Wazuh.
- Created custom Wazuh escalation Rule 100110 (Level 12).
- Validated positive and negative HTTP detection tests.
- Confirmed automatic startup and low memory consumption.
- Removed temporary NAT rules and restored network isolation.

**Result:** Custom Suricata alerts successfully detected and escalated in Wazuh.

## Task 4 — SCA and Vulnerability Assessment

| Endpoint | Passed | Failed | N/A | Score |
|---|---:|---:|---:|---:|
| Ubuntu 24.04 | 119 | 118 | 42 | 50% |
| Windows 10 | 116 | 303 | 5 | 27% |

The table represents the initial SCA baseline.

Ubuntu security hardening:
- CIS Check 35547: Failed to Passed.
- CIS Check 35548: Failed to Passed.
- CIS Check 35510: Documented risk acceptance.

The top three CVSS-ranked vulnerability findings were collected for both endpoints. No vulnerability patches were applied.

## Repository Structure

- rules/ — Custom Wazuh decoder and detection rules
- evidence/ — Security testing and verification screenshots
- architecture/ — Network architecture and addressing
- journal.md — Laboratory activities and troubleshooting
- README.md — Project documentation

## Recovery and Rollback

### Wazuh Service Recovery

On the SIEM host:

    cd ~/wazuh-docker/single-node
    sudo docker compose ps
    sudo docker compose up -d

Persistent Docker volumes retain indexed security events.

### Custom Detection Rule Recovery

Restore custom configuration files from this repository if missing after container recreation:

    cd ~/mini-soc-week2

    sudo docker cp rules/labshop_decoder.xml single-node-wazuh.manager-1:/var/ossec/etc/decoders/labshop_decoder.xml

    sudo docker cp rules/labshop_rules.xml single-node-wazuh.manager-1:/var/ossec/etc/rules/labshop_rules.xml

    sudo docker cp rules/suricata_rules.xml single-node-wazuh.manager-1:/var/ossec/etc/rules/suricata_rules.xml

Validate the configuration:

    sudo docker exec single-node-wazuh.manager-1 /var/ossec/bin/wazuh-analysisd -t

After successful validation:

    cd ~/wazuh-docker/single-node
    sudo docker compose restart wazuh.manager

### Endpoint Recovery

- Ubuntu agent configuration: /var/ossec/etc/ossec.conf
- Windows agent configuration: C:\Program Files (x86)\ossec-agent\ossec.conf
- Suricata configuration: /etc/suricata/suricata.yaml
- Suricata signatures: /var/lib/suricata/rules/local.rules

Restore endpoint configurations from verified backups or VM snapshots when required.

### Login Banner Rollback

Original Ubuntu login banners are preserved in:

    /etc/issue.labshop.bak
    /etc/issue.net.labshop.bak

### Network Isolation

Endpoint default gateways and temporary installation NAT rules were removed after Suricata installation.

Do not run `docker compose down -v`, as it removes persistent volumes.

## Project Status

Week 2 Tasks 1–4: Completed.

All security testing was performed within an authorized, isolated laboratory environment.
