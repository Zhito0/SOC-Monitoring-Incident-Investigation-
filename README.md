# SOC-Monitoring-Incident-Investigation-
SOC Analyst portfolio project using Wazuh, Ubuntu, Kali Linux, and VirtualBox to monitor security events, investigate alerts, and document incident response

# SOC Monitoring and Incident Investigation Lab

## Project Overview

This project demonstrates practical Security Operations Center (SOC) analyst skills through security monitoring, alert triage, incident investigation and evidence-based reporting.

The environment uses Wazuh SIEM/XDR, Ubuntu Linux, Kali Linux and Oracle VirtualBox.

## Project Objectives

- Deploy a working security monitoring platform.
- Connect and monitor Linux endpoints.
- Investigate security alerts using Wazuh.
- Distinguish false positives from potentially malicious activity.
- Document investigation findings and recommended actions.

## Lab Architecture

| Component | Purpose |
|---|---|
| Wazuh Server | Security monitoring and alert analysis |
| Ubuntu Linux | Monitored endpoint |
| Kali Linux | Controlled security testing |
| VirtualBox | Virtualization and isolated networking |

## Completed Investigation

### Case Study 001 — Wazuh Rootcheck Alert

Investigated a medium-severity Rootcheck alert (Rule 510) flagging `/usr/bin/md5sum` as potentially trojaned.

The investigation included package ownership verification, package integrity checking and executable-path analysis.

**Disposition:** Likely false positive. No confirmed compromise.

Full report: [Read Case Study 001](investigations/case-001-rootcheck-false-positive.md)

## Skills Demonstrated

- SIEM deployment and monitoring
- Endpoint security monitoring
- Security alert triage
- Linux command-line investigation
- Package integrity verification
- Evidence-based incident reporting

## Project Status

SOC monitoring environment operational. Additional controlled security investigations planned.
