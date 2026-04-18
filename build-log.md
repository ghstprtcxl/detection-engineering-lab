# Detection Engineering Lab
**Author:** ghstprtcxl  
**Type:** Active Directory adversary simulation + SIEM detection pipeline  
**Stack:** Proxmox, pfSense, Windows Server 2022, Wazuh, Sysmon, Kali Linux  

## Overview
Isolated, instrumented Active Directory environment designed to generate 
realistic attack telemetry and validate detection logic. Built on constrained 
hardware (16 GB RAM) with full network segmentation between attacker, victim, 
and monitoring planes.

## Key Capabilities Demonstrated
- **Network Architecture:** Multi-segment VLAN design with firewall-enforced isolation
- **Log Ingestion:** Custom Wazuh agent groups with Sysmon Event ID correlation
- **Attack Simulation:** Kerberoasting, LDAP enumeration, lateral movement prep
- **Detection Validation:** End-to-end alert generation from offensive action to SIEM dashboard

&gt; **Design Note:** RAM budget forced a CLI-only Kali deployment and a cold-spare Debian management VM. Future expansion to 32 GB will allow Security Onion and a web server.

---

## 4. Build Log

### Day 1 — Friday, April 17, 2026
**Focus:** Infrastructure wipe, network segmentation, and pfSense baseline.

- Wiped Proxmox node for clean build.
- Configured four virtual bridges (vmbr0-3).
- Deployed pfSense with quad-NIC setup:
  - WAN → vmbr0
  - LAN → vmbr1 (later reassigned or clarified in Day 2)
  - OPT1 → vmbr2
  - OPT2 → vmbr3
- Enabled DHCP on all internal segments (.100 – .200).
- Deployed Debian management VM on LAN for pfSense Web GUI access.
- Drafted initial firewall rules:
  - [Document Attack_Net rules]
  - [Document Victim_Net rules]
  - [Document Wazuh_Net rules]

### Day 2 — Saturday, April 18, 2026
**Focus:** Active Directory population and Wazuh ingestion.

- **BadBlood execution:** Generated 500 users, 2,500 groups, 50 computers on DC01.
- **Sysmon deployment:**
  - Generated config via Sysmon-Modular.
  - Manual file copy required due to CLI execution issues on DC.
- **Wazuh agent installation (DC01):**
  - Initial config pointed to wrong server IP (DC instead of Wazuh manager).
  - Troubleshot `Invoke-WebRequest` syntax error — used PowerShell instead of cmd.
  - Added DC to custom Wazuh group with Sysmon log ingestion rules.
  - Verified log flow in `/var/ossec/log/ossec.log` and Wazuh Dashboard.
  - Added firewall syslog rule to forward pfSense events to Wazuh.
- **First detection:** PowerShell script execution (Sysmon Event ID 1) observed in dashboard.

### Day 3 — Saturday, April 18, 2026 (Evening)
**Focus:** Attack infrastructure and file server deployment under 16 GB constraint.

- **Decision:** Kali Linux deployed as CLI-only (1.5 GB / 1 vCPU) to preserve RAM.
- [To be filled: File Server build, domain join, audit policy]
- [To be filled: Kali SSH hardening and tool installation]
- [To be filled: Attack simulations — SharpHound, Kerberoasting]
- [To be filled: Wazuh detection validation]

---

## 5. Current Issues & Lessons Learned
| Issue | Resolution / Workaround |
|-------|-------------------------|
| 16 GB RAM insufficient for GUI Kali + all services | Kali switched to CLI-only (`multi-user.target`), saving ~1 GB |
| Wazuh agent pointed to DC IP instead of manager | Manually edited `ossec.conf` server IP entries |
| `Invoke-WebRequest` failed in cmd | Ran command in PowerShell host instead |
| Sysmon-Modular CLI generation failed | Copied generated `.xml` manually to target |

---

## 6. Next Steps / Roadmap
- [ ] Complete File Server deployment (Win Srv 2022, 2 GB) with file audit policies
- [ ] Execute SharpHound / BloodHound.py from Kali → verify Wazuh catches LDAP enumeration
- [ ] Kerberoasting simulation → confirm Event ID 4769 detection
- [ ] Deploy internal web server (Victim_NET) — pending RAM upgrade
- [ ] Deploy Security Onion sensor — requires hardware rebalancing or node addition
- [ ] Inline IPS (pfSense Suricata or dedicated sensor)

---

## 7. References
- BadBlood: https://github.com/davidprowe/BadBlood
- Sysmon-Modular: https://github.com/olafhartong/sysmon-modular
- Wazuh Documentation: https://documentation.wazuh.com