# SOC Case Study 003: File Integrity Monitoring and Tampering Detection

## Executive Summary

Conducted a controlled file-integrity monitoring exercise on an Ubuntu endpoint connected to Wazuh SIEM.

Created a test file within a monitored directory and subsequently modified its contents. Wazuh detected both the file creation and modification, generating security alerts with corresponding file-integrity evidence.

**Disposition:** True positive detection — authorized laboratory activity.

## Lab Environment

| Component | Description |
|---|---|
| SIEM | Wazuh 4.14.9 |
| Endpoint | Ubuntu Linux |
| Agent | ubuntu-soc-endpoint |
| Endpoint IP | 192.168.56.2 |
| Manager IP | 192.168.56.4 |
| Monitoring module | Wazuh File Integrity Monitoring (Syscheck) |
| Virtualization | Oracle VirtualBox |

## Investigation Objective

Validate Wazuh's ability to detect file creation and modification, identify changed file attributes, and support security investigations through checksum evidence.

## Test Configuration

Configured real-time monitoring for:

`/home/vboxuser/soc-fim-lab`

Using the Wazuh agent's `ossec.conf` configuration:

```xml
<directories realtime="yes" check_all="yes" report_changes="yes">/home/vboxuser/soc-fim-lab</directories>
```

## Test Procedure

### 1. Create the test file

```bash
echo "SOC Lab - Original file content" > /home/vboxuser/soc-fim-lab/test-file.txt
```

### 2. Modify the file

```bash
echo "SOC Lab - File modified for security investigation" >> /home/vboxuser/soc-fim-lab/test-file.txt
```

### 3. Investigate Wazuh alerts

Opened Wazuh File Integrity Monitoring → Events and reviewed the alerts associated with `test-file.txt`.

## Detection Results

| Attribute | File creation | File modification |
|---|---|---|
| Wazuh rule ID | 554 | 550 |
| Rule level | 5 | 7 |
| Event type | added | modified |
| Endpoint | ubuntu-soc-endpoint | ubuntu-soc-endpoint |

## Evidence Analysis

The Rule 550 event reported:

- Monitored file: `/home/vboxuser/soc-fim-lab/test-file.txt`
- Monitoring mode: Real-time
- File size before: 31 bytes
- File size after: 81 bytes
- Changed attributes: Size, modification time, MD5, SHA-1 and SHA-256

The recorded checksums changed following the modification, supporting the conclusion that Wazuh detected a content change.

## Assessment

**Classification: True Positive — Authorized Activity**

The FIM alert accurately reflected an intentional modification performed in a controlled laboratory environment.

No unauthorized tampering was established.

## Recommendations

- Enable FIM on critical directories and files where appropriate.
- Investigate unexpected checksum or permission changes.
- Correlate FIM alerts with system authentication and process activity.
- Maintain secure baselines for monitored files.
- Restrict and audit administrative changes to sensitive files.

## Skills Demonstrated

- Real-time File Integrity Monitoring configuration
- Endpoint security monitoring
- File-change detection
- Checksum-based evidence analysis
- SIEM alert triage and classification
- SOC investigation documentation

## Conclusion

Wazuh successfully detected both the creation and modification of a monitored file.

The exercise confirmed the effectiveness of real-time file integrity monitoring for identifying changes that may require security investigation.
