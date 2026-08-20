# Day 16 — Blue Team & SOC Fundamentals

**Date:** Aug 17, 2026
**Week:** Week 4 — Defense, Mobile & Capstone

---

## 🎯 Overview

Switching sides — Week 4 opens with defensive fundamentals and standing up a real SIEM.
Every offensive technique from Weeks 2-3 now becomes something to *detect*, not perform.

## 🛡️ Defense-in-Depth

The principle of layering multiple, independent security controls so that a single
failure doesn't mean full compromise:

```
Perimeter (firewall) → Network (segmentation, IDS) → Host (EDR, hardening)
   → Application (input validation, Day 7-8) → Data (encryption, Day 3) → User (training)
```
No single layer is assumed sufficient — this is *why* a real environment needs the SIEM
being built today: it's the layer that correlates signals across all the others.

## 📊 SIEM Concepts

A SIEM (Security Information and Event Management) system centralizes security-relevant
events from many sources so they can be correlated and alerted on.

```
Log sources (endpoints, firewalls, apps)
      ↓  ingestion
Normalization  (different log formats → one common schema)
      ↓
Correlation    (rules that connect events across sources/time)
      ↓
Alerting        (notify a human when a rule matches)
```
**Why correlation matters:** a single failed login is noise. 50 failed logins from one IP
in 60 seconds followed by one success is a brute-force attack — that pattern only becomes
visible when a SIEM correlates events over time, which a human staring at one log file
never would in real time.

## 🐺 Deploying Wazuh

Wazuh has two main components: the **manager** (central server: rules engine, dashboard,
API) and **agents** (installed on each monitored endpoint, forwarding logs/telemetry back).

```bash
# Manager install (Debian/Ubuntu, official repo)
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a          # installs manager + indexer + dashboard

systemctl status wazuh-manager          # confirm it's running
```

## 🖇️ Enrolling Agents

```bash
# On the manager: generate an enrollment key for a new agent
/var/ossec/bin/manage_agents -a

# On the endpoint being monitored: install the agent, point it at the manager
WAZUH_MANAGER="manager_ip" apt-get install wazuh-agent
systemctl start wazuh-agent
```
Once enrolled, an agent's logs (file integrity, process, log-file monitoring) start
flowing into the manager and are visible in the dashboard within a minute or two.

## ✍️ Writing a Custom Alert Rule

Wazuh ships with a large default ruleset, but the highest-value exercise is writing a
custom rule for something specific — like **Day 14**'s persistence artifact.

```xml
<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="local,custom_persistence,">
  <rule id="100010" level="12">
    <if_sid>530</if_sid>  <!-- base: registry change event -->
    <field name="win.eventdata.targetObject">CurrentVersion\\Run</field>
    <description>Possible persistence via Registry Run key</description>
    <mitre>
      <id>T1547.001</id>
    </mitre>
  </rule>
</group>
```
Restarting the manager (`systemctl restart wazuh-manager`) loads the new rule. Triggering
Day 14's benign loader again on an enrolled endpoint should now produce a real alert in
the dashboard — closing the loop from "built the attack" to "built the detection."

## 🔗 How This Connects

- The custom alert rule here should fire on the exact persistence artifact built on
  **Day 14** — the attacker→defender loop closes for real, not just conceptually.
- SIEM correlation concepts here are what makes **Day 17**'s raw log timeline actually
  scale beyond manually reading one file.
- This is the mental "switch sides" moment: every Week 2-3 technique from here forward is
  read as something to detect.

## ✅ Checkpoint / Deliverable

- [ ] Wazuh deployment with at least one enrolled agent, confirmed reporting in the dashboard
- [ ] One custom alert rule written and confirmed to fire correctly against a real trigger
