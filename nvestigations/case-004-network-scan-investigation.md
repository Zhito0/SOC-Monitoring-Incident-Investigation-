**SOC Case Study 004: Suspicious Network Activity Detection and Firewall Alert Engineering**
1. Executive Summary
This case study documents a controlled network security investigation conducted using Kali Linux, Ubuntu Linux, and Wazuh SIEM.
A targeted Nmap TCP connection scan was performed from Kali Linux against an Ubuntu endpoint within an isolated VirtualBox laboratory environment.
Ubuntu's Uncomplicated Firewall (UFW) successfully blocked connections to restricted ports and recorded the activity in its kernel journal.
During the investigation, no matching Wazuh alerts were initially identified for the Kali scan. A custom Wazuh detection rule was subsequently developed, tested, and activated.
A separate controlled connection attempt from the MacBook host was used to validate the live detection pipeline. Wazuh successfully generated five matching firewall alerts.
Investigation outcome: Successful network security monitoring and custom detection-rule validation.
Final classification: True positive detection — authorized laboratory activity.
Incident status: No confirmed malicious activity or system compromise.

2. Investigation Objectives
The objectives were to:
    •    Perform controlled network reconnaissance against an authorized endpoint.
    •    Identify open and filtered TCP ports.
    •    Examine Ubuntu firewall logs for blocked connections.
    •    Determine whether Wazuh was detecting the firewall activity.
    •    Develop and test a custom firewall detection rule.
    •    Validate live alert generation.
    •    Document the investigation using SOC analysis procedures.

3. Laboratory Environment
Component
Description
SIEM
Wazuh 4.14.9
Hypervisor
Oracle VirtualBox
Operating environment
macOS host
Scanning machine
Kali Linux
Kali IP address
192.168.56.3
Target operating system
Ubuntu Linux
Ubuntu endpoint IP
192.168.56.2
Wazuh agent name
ubuntu-soc-endpoint
Wazuh manager IP
192.168.56.4
MacBook validation IP
192.168.56.1
Firewall
UFW
Network configuration
VirtualBox host-only network
All simulated traffic was restricted to systems within the authorized laboratory.

4. Phase One: Firewall Configuration Verification
The Ubuntu firewall configuration was examined using:
sudo ufw status verbose
Findings
    •    Firewall status: Active
    •    Logging: On (low)
    •    Default incoming policy: Deny
    •    Default outgoing policy: Allow
    •    Default routed policy: Deny
    •    HTTP port 80: Allowed from 192.168.56.0/24
The configuration established a baseline for interpreting the results of the upcoming network scan.
Evidence: 01-ubuntu-firewall-status.png

5. Phase Two: Controlled Network Scan
A TCP connection scan was performed from Kali Linux against the Ubuntu endpoint.
Command
nmap -sT -Pn -p 21,22,23,80,443,8080 --max-retries 1 192.168.56.2
Command explanation
    •    -sT: Perform a TCP connection scan.
    •    -Pn: Skip host discovery.
    •    -p: Specify the selected destination ports.
    •    --max-retries 1: Limit probe retransmissions.
Nmap results
Port
Service label
Result
21/tcp
FTP
Filtered
22/tcp
SSH
Filtered
23/tcp
Telnet
Filtered
80/tcp
HTTP
Open
443/tcp
HTTPS
Filtered
8080/tcp
HTTP proxy
Filtered
Total ports scanned: 6
Open ports: 1
Filtered ports: 5
Scan duration: 14.31 seconds
Analysis
The results indicated that Ubuntu was reachable and accepting TCP connections on port 80.
The other five tested ports appeared filtered.
These findings were consistent with the firewall's incoming traffic policy and its explicit allowance for HTTP traffic.
Evidence: 02-kali-nmap-scan-results.png

6. Phase Three: Firewall Log Investigation
Ubuntu's kernel journal was examined for blocked network traffic.
Command
sudo journalctl -k --no-pager | grep -i 'UFW' | tail -n 20
Findings
The firewall recorded blocked TCP connection attempts from Kali Linux.
Important log fields included:
Field
Description
UFW BLOCK
Firewall blocking action
SRC
Source IP address
DST
Destination IP address
PROTO
Network protocol
SPT
Source port
DPT
Destination port
SYN
TCP connection initiation flag
The observed entries included:
    •    Source IP: 192.168.56.3
    •    Destination IP: 192.168.56.2
    •    Protocol: TCP
    •    Destination ports: 21, 22, 23, 443, 8080
Analysis
The firewall evidence correlated with the filtered ports reported by Nmap.
This established that the Kali scan reached Ubuntu and that UFW blocked the restricted connection attempts.
Evidence: 03-ubuntu-ufw-block-logs.png

7. Phase Four: Initial Wazuh Investigation
Wazuh Threat Hunting was used to search for firewall alerts.
The initial search did not return matching results.
A subsequent inspection of the manager's alert records also failed to identify an alert containing Kali's source IP.
Analysis
The absence of matching alerts indicated a visibility gap in the initial investigation.
Further testing was required to distinguish between log collection issues and detection-rule limitations.
The firewall events were present in Ubuntu's kernel journal, but Wazuh detection of the original Kali scan was not established.

8. Phase Five: Wazuh Logtest Analysis
Wazuh's Logtest utility was used to examine how the detection engine processed a representative firewall log.
Command
sudo /var/ossec/bin/wazuh-logtest
Findings
The initial processing identified:
Phase 1 — Pre-decoding
    •    Hostname: Ubuntu
    •    Program: kernel
    •    Timestamp: Parsed
Phase 2 — Decoding
    •    Decoder: kernel
An alert-generating firewall rule was not initially observed for the sample event.
This prompted the development of a custom UFW detection rule.

9. Phase Six: Custom Wazuh Rule Development
A new rule was added to the Wazuh manager's local rules configuration.
File location:
/var/ossec/etc/rules/local_rules.xml
Custom Rule 100100
<group name="firewall,ufw,">
  <rule id="100100" level="5">
    <program_name>kernel</program_name>
    <match>UFW BLOCK</match>
    <description>Ubuntu UFW blocked a network connection</description>
  </rule>
</group>
Rule explanation
Component
Purpose
Rule ID
Unique identifier 100100
Rule level
Severity level 5
Program name
Match kernel log messages
Match condition
Detect UFW BLOCK messages
Description
Identify blocked network connections
Groups
firewall, ufw
The purpose of the rule was to identify blocked firewall connections.
It was not designed to independently establish that a port scan or malicious intrusion occurred.
Evidence: 04-custom-wazuh-firewall-rule.png

10. Phase Seven: Custom Rule Validation
Wazuh Logtest was executed again using a sample firewall message.
Test result
The detection engine successfully matched the custom rule.
Phase 1: Completed pre-decoding

Phase 2: Completed decoding
name: kernel

Phase 3: Completed filtering (rules)
id: 100100
level: 5
description: Ubuntu UFW blocked a network connection

Alert to be generated.
Analysis
The successful Logtest result established that Wazuh's detection engine could recognize the representative UFW message.
However, Logtest alone does not prove live event collection. Additional validation was necessary.
Evidence: 05-wazuh-logtest-rule-success.png

11. Phase Eight: Live Firewall Detection Validation
The Wazuh manager was restarted to activate the custom detection rule.
Command
sudo systemctl restart wazuh-manager
Manager status was verified using:
sudo systemctl is-active wazuh-manager
Result: Active
A separate controlled connection attempt was made from the MacBook host to Ubuntu's TCP port 22.
The connection timed out, and Ubuntu recorded corresponding UFW blocking events.
Live event details
Attribute
Observed value
Source IP
192.168.56.1
Destination IP
192.168.56.2
Protocol
TCP
Destination port
22
Firewall action
UFW BLOCK
Wazuh validation
The manager's recorded alerts were examined using:
sudo grep '"id":"100100"' /var/ossec/logs/alerts/alerts.json | tail -n 5
The command returned live alerts associated with Rule 100100.
This confirmed that the monitoring pipeline could collect, process and alert on real firewall activity.
Evidence: 06-live-wazuh-firewall-alerts.png

12. Phase Nine: Wazuh Dashboard Investigation
Wazuh Threat Hunting was used to investigate the live alerts.
Search query
rule.id: 100100
Dashboard results
Total matching alerts: 5
Expanded alert findings
Field
Value
Agent
ubuntu-soc-endpoint
Agent IP
192.168.56.2
Rule ID
100100
Rule description
Ubuntu UFW blocked a network connection
Rule level
5
Log source
journald
Decoder
kernel
Rule groups
firewall, ufw
Source IP
192.168.56.1
Destination IP
192.168.56.2
Destination port
22
Protocol
TCP
Event correlation
The five dashboard alerts were associated with the later MacBook connection-validation activity.
The earlier Kali Nmap scan was independently confirmed in Ubuntu's firewall journal.
These observations represent two distinct stages of the investigation and should not be treated as the same event.
Evidence: 07-wazuh-rule-100100-dashboard.png
Evidence: 08-wazuh-firewall-alert-details.png

13. SOC Analyst Assessment
Detection classification
True positive — authorized laboratory activity
The firewall accurately recorded blocked network connections.
Wazuh successfully detected the live validation traffic after deployment of the custom rule.
Security impact
No evidence was established of:
    •    Unauthorized system access
    •    Successful exploitation
    •    Credential compromise
    •    Data exfiltration
    •    System compromise
Investigation limitations
The original Kali scan was confirmed through Ubuntu firewall logs, but matching live Wazuh alerts from that original scan were not established.
The custom rule detects individual UFW blocking events. It does not automatically identify a multi-port scan.
The exercise also revealed differences between virtual-machine log timestamps and Wazuh indexed timestamps. Clock synchronization should be reviewed before constructing a precise forensic timeline.

14. Recommendations
    1    Maintain firewall logging on monitored Linux endpoints.
    2    Ensure the Wazuh agent collects relevant kernel and UFW events.
    3    Apply custom detection rules where justified by monitoring requirements.
    4    Validate new rules through both Logtest and live event generation.
    5    Monitor repeated blocked connections from the same source.
    6    Correlate activity across ports and time windows before classifying behaviour as network scanning.
    7    Synchronize system clocks across security monitoring systems.
    8    Review and tune alert thresholds to reduce unnecessary security noise.
    9    Preserve relevant security evidence for subsequent investigation.

15. Skills Demonstrated
    •    SOC security monitoring
    •    Network reconnaissance investigation
    •    Nmap TCP scanning
    •    Linux UFW analysis
    •    Kernel journal investigation
    •    Security event correlation
    •    Wazuh custom detection engineering
    •    XML security rule configuration
    •    Wazuh Logtest validation
    •    Live SIEM alert verification
    •    Security incident classification
    •    Evidence-based SOC reporting

16. Conclusion
This investigation successfully demonstrated a complete security monitoring workflow using Kali Linux, Ubuntu, UFW and Wazuh SIEM.
The controlled Kali network scan produced observable firewall blocking events.
A custom Wazuh detection rule was developed and successfully validated against live firewall traffic generated during a separate test.
The investigation reinforced the distinction between recording a network event, detecting it through a SIEM, and determining whether the activity represents an actual security threat.
Final result: Successful detection engineering and authorized network security investigation.
Case Study 004 Status: Completed.
