# SOC Case Study 001: Investigation of a Suspected Trojaned Linux Binary

## Executive Summary

Investigated a medium-severity Wazuh Rootcheck alert that identified `/usr/bin/md5sum` as potentially trojaned on an Ubuntu endpoint.

Package ownership, executable-path verification, and Debian package integrity checks found no discrepancies. The findings are consistent with a documented Wazuh false-positive issue affecting Rust-based coreutils.

**Disposition:** Likely false positive — no confirmed compromise.

## Lab Environment

- SIEM: Wazuh 4.14.9
- Monitored endpoint: `ubuntu-soc-endpoint`
- Endpoint IP: `192.168.56.2`
- Wazuh manager: `192.168.56.4`
- Virtualization: VirtualBox
- Network: Isolated host-only lab

## Alert Details

| Field | Observed value |
|---|---|
| Detection date | November 2, 2026 |
| Rule ID | 510 |
| Rule level | 7 — Medium |
| Detection module | Rootcheck |
| Alert description | Host-based anomaly detection event |
| Flagged file | `/usr/bin/md5sum` |
| Detection message | Trojaned version of file detected |

## Investigation Procedure

### 1. Identify the package owner

Command:

`dpkg -S /usr/bin/md5sum`

Result:

`coreutils-from-uutils: /usr/bin/md5sum`

### 2. Verify the owning package

Command:

`sudo dpkg -V coreutils-from-uutils`

Result: No discrepancies reported.

### 3. Resolve the executable path

Command:

`readlink -f /usr/bin/md5sum`

Result:

`/usr/lib/cargo/bin/coreutils/md5sum`

### 4. Verify the Rust coreutils package

Command:

`sudo dpkg -V rust-coreutils`

Result: No discrepancies reported.

## Analysis

The flagged binary is associated with the Rust-based coreutils implementation.

The package integrity checks reported no discrepancies. A publicly documented Wazuh issue describes similar false-positive Rule 510 detections involving Rust coreutils and `/usr/bin/md5sum`.

These findings support a likely false-positive classification. However, package verification alone cannot establish that a system is entirely free of malware.

## Conclusion

**Disposition: Likely False Positive**

The evidence did not establish malicious modification of `/usr/bin/md5sum`.

No destructive remediation was performed. The file and its packages were left unchanged to preserve system integrity.

## Recommendations

- Continue monitoring the endpoint for additional suspicious activity.
- Review future Rule 510 detections individually.
- Monitor official Wazuh updates addressing Rust-coreutils signature false positives.
- Avoid disabling Rule 510 globally without assessing the impact on detection coverage.
- Reassess the case if new evidence suggests unauthorized file modification.

## Evidence

- Wazuh Threat Hunting dashboard showing Rule 510.
- Expanded alert details identifying `/usr/bin/md5sum`.
- Ubuntu terminal showing package ownership.
- Ubuntu terminal showing successful package verification checks.

## Reference

Wazuh GitHub Issue #39223:
https://github.com/wazuh/wazuh/issues/39223
