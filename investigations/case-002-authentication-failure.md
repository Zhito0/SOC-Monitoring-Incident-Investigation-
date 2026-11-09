# SOC Case Study 002: Authentication Failure Detection and Investigation

## Executive Summary

Conducted a controlled authentication-failure simulation on an Ubuntu endpoint monitored by Wazuh SIEM.

An intentionally incorrect `sudo` password generated authentication-failure logs. Wazuh collected the events and produced two related security alerts.

**Disposition:** Authorized security test — no confirmed malicious activity.

## Lab Environment

| Component | Description |
|---|---|
| SIEM | Wazuh 4.14.9 |
| Endpoint | Ubuntu Linux |
| Agent name | ubuntu-soc-endpoint |
| Agent IP | 192.168.56.2 |
| Wazuh manager | 192.168.56.4 |
| Network | VirtualBox host-only network |

## Investigation Objective

Evaluate whether Wazuh can detect failed authentication attempts, identify the associated account, and provide sufficient evidence for security triage.

## Simulation Procedure

1. Cleared the cached sudo authentication using `sudo -k`.
2. Executed `sudo whoami`.
3. Entered one intentionally incorrect password.
4. Entered the correct password on the next prompt.
5. Verified the authentication failure in Ubuntu system logs.
6. Investigated the related events through Wazuh Threat Hunting.

## Wazuh Detection Results

| Attribute | Alert 1 | Alert 2 |
|---|---|---|
| Rule ID | 5503 | 5557 |
| Description | PAM: User login failed | unix_chkpwd: Password check failed |
| Severity | Level 5 | Level 5 |
| Agent | ubuntu-soc-endpoint | ubuntu-soc-endpoint |
| Account | vboxuser | vboxuser |
| MITRE mapping | T1110.001 | T1110.001 |

## Evidence Analysis

### Alert 1 — Rule 5503

The PAM authentication system recorded a failed authentication attempt involving `sudo`.

Relevant log:

`pam_unix(sudo:auth): authentication failure`

### Alert 2 — Rule 5557

The password-checking component recorded a corresponding password verification failure.

Relevant log:

`unix_chkpwd: password check failed for user (vboxuser)`

Both alerts correspond to the same account and original event time, supporting correlation with the controlled test.

## Timeline Observation

The Ubuntu log recorded the authentication failure at approximately November 9, 2026, 21:14:13, while Wazuh displayed indexed timestamps around 16:42:36.

This discrepancy should be investigated before using the timestamps for precise incident reconstruction.

## MITRE ATT&CK Mapping

- **Tactic:** Credential Access
- **Technique:** T1110.001 — Password Guessing

The mapping identifies the type of behaviour associated with the detection rule. It does not establish that an attacker performed password guessing.

## Assessment and Disposition

**Classification:** Authorized test activity.

The authentication failure was deliberately generated in a controlled laboratory environment. The associated Wazuh alerts demonstrate that the monitoring system captured and classified the activity.

No evidence of malicious access was established by this investigation.

## Recommended Actions

- Continue monitoring authentication events.
- Maintain centralized Linux authentication logging.
- Investigate repeated failures, especially across multiple accounts or unusual sources.
- Review timestamp synchronization between the endpoint and Wazuh.
- Preserve relevant evidence when investigating genuine suspicious activity.

## Skills Demonstrated

- Security event generation
- Linux authentication-log analysis
- SIEM alert triage
- Wazuh rule correlation
- MITRE ATT&CK interpretation
- Evidence-based security reporting

## Conclusion

The controlled simulation successfully demonstrated authentication-failure detection and investigation using Wazuh SIEM.

The two related alerts were classified as expected test activity, with no confirmed compromise.
