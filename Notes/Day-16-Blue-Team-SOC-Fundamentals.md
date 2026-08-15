# Day 16 — Blue Team & SOC Fundamentals

**Date:** Aug 17, 2026  
**Week:** Week 4: Defense, Mobile & Capstone  

---

## 🎯 Overview

Switching sides — Week 4 opens with defensive fundamentals and standing up a real SIEM.

## 📚 Topics Covered

- Defense-in-depth as a layered security model
- SIEM concepts: log ingestion, normalization, correlation, alerting
- Deploying Wazuh (manager + dashboard)
- Enrolling agents on lab hosts so their logs flow into the SIEM
- Writing a custom Wazuh alert rule for one of Week 3's malware artifacts

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Wazuh** | Open-source SIEM/XDR platform |
| **Wazuh agent** | Endpoint log/telemetry collection |
| **Wazuh Ruleset** | Custom detection rule authoring |

## 💻 Key Commands

```bash
/var/ossec/bin/manage_agents
systemctl status wazuh-manager
custom rule in local_rules.xml matching a Week 3 IOC
```

## 🔗 How This Connects

- The custom alert rule here should fire on the exact persistence/injection artifact built on Day 14 — closing the loop attacker→defender.
- SIEM correlation concepts here are what makes Day 17's raw log timeline actually scale.
- This is the mental 'switch sides' moment: every Week 2-3 technique is now something to detect, not perform.

## ✅ Checkpoint / Deliverable

- [ ] Wazuh deployment with at least one enrolled agent and one custom alert rule that fires correctly.

---
