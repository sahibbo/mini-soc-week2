# Week 2 — Mini-SOC Lab Journal

**Project:** LabShop Mini-SOC
**Platform:** Wazuh 4.14.8
**Environment:** PNETLab
**Period:** October 2026

---

## Entry 01 — Custom Decoder and Alert Tuning

**Task:** Implement LabShop authentication monitoring.

**Challenge:** Expected authentication events from the monitoring account could generate unnecessary security alerts.

**Resolution:**
- Created a custom PCRE2 decoder.
- Implemented failed-login Rule 100100.
- Excluded the monitor account from custom detection.
- Verified the exclusion using Wazuh Logtest.

**Result:** Failed authentication detection worked without unnecessary custom alerts from the monitoring account.

---

## Entry 02 — Brute-Force Correlation

**Task:** Detect repeated authentication failures.

**Challenge:** Failed logins from different source IP addresses must not be combined into a single brute-force alert.

**Resolution:**
- Created correlation Rule 100101.
- Configured frequency 5 and timeframe 60 seconds.
- Applied same-source-IP correlation.
- Tested matching and different source IP addresses.

**Result:** Level 12 brute-force detection successfully validated and mapped to MITRE ATT&CK T1110.

---

## Entry 03 — File Integrity Monitoring

**Task:** Monitor Linux and Windows file changes.

**Challenge:** Scheduled FIM scans originally operated at long intervals, making verification time-consuming.

**Resolution:**
- Enabled real-time FIM for Linux and Windows test directories.
- Temporarily reduced scheduled scan intervals to 60 seconds.
- Tested file creation, modification, deletion, and Windows Registry changes.
- Inspected syscheck.diff for content differences.

**Result:** Real-time and scheduled FIM alerts were successfully detected. Precise latency comparison was not established because of clock synchronization differences.

---

## Entry 04 — Suricata Installation and Network Rollback

**Task:** Deploy Suricata IDS on the isolated Ubuntu endpoint.

**Challenge:** The endpoint lacked direct internet access. Installation also required resolving repository key issues and correcting the monitored network interface.

**Resolution:**
- Temporarily configured NAT forwarding through siem-host.
- Resolved Ubuntu repository key verification issues.
- Installed Suricata 7.0.3.
- Corrected the capture interface to ens3.
- Created HTTP detection signature 1000001.
- Removed temporary NAT, forwarding, DNS, and default-route changes after installation.

**Result:** Suricata operated successfully while the original isolated SOC-LAN networking was restored.

---

## Entry 05 — Suricata Alert Escalation

**Task:** Integrate Suricata alerts into Wazuh.

**Challenge:** The default Suricata alert severity did not represent the desired priority for LabShop admin-path access.

**Resolution:**
- Configured Wazuh Agent collection of Suricata EVE JSON logs.
- Created Wazuh Rule 100110 with severity Level 12.
- Validated the custom rule using wazuh-logtest.
- Performed positive and negative HTTP detection tests.

**Result:** Requests targeting /admin generated Level 12 alerts. Normal HTTP requests did not trigger the custom detection.

---

## Entry 06 — Security Assessment and Hardening

**Task:** Assess endpoint security and remediate selected CIS findings.

**Challenge:** Ubuntu and Windows endpoints contained multiple failed CIS checks. Remediating all findings could introduce unnecessary operational risk.

**Resolution:**
- Collected Ubuntu and Windows SCA baselines.
- Remediated Ubuntu CIS Checks 35547 and 35548.
- Verified both checks changed from Failed to Passed.
- Documented risk acceptance for Check 35510.
- Reviewed the top three CVSS-ranked vulnerability findings on each endpoint.

**Result:** Two Ubuntu security configuration findings were successfully remediated, and vulnerability inventories were documented.

---

## Week 2 Lessons Learned

- Custom detection rules require positive and negative testing.
- Source IP correlation reduces incorrect brute-force alerts.
- File Integrity Monitoring provides visibility into endpoint modifications.
- IDS alerts can be prioritized using custom Wazuh rules.
- Temporary network changes must be rolled back after installation.
- Security hardening should balance compliance and operational risk.
- Vulnerability assessment findings require prioritization and validation.

---

## Entry 07 — Container Recreation and Persistence

**Task:** Verify custom Wazuh rule persistence after container recreation.

**Challenge:** Custom detection files must survive container replacement without losing security configurations.

**Resolution:**
- Verified the persistent Docker volume mounted at /var/ossec/etc.
- Recreated only the Wazuh Manager container.
- Confirmed all three custom XML files remained available.
- Validated Wazuh configuration with exit code 0.
- Confirmed Manager, Indexer, and Dashboard were running.

**Result:** Custom Wazuh configurations successfully survived container recreation.
