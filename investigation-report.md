
# pfSense Security Lab — SOC Investigation Report

## 1. Executive Summary

This investigation examines controlled network connection attempts between Kali Linux and an Ubuntu web server across a pfSense firewall. The objective was to analyze firewall decisions, identify TCP retransmissions, and correlate pfSense logs with Wireshark packet captures.

The activity was intentionally generated within an authorized VirtualBox homelab. It does not represent a real-world security incident.

## 2. Lab Environment

- Firewall: pfSense 2.9.0
- Kali Linux: 192.168.60.100/24 (TestNet)
- Ubuntu Server: 192.168.50.100/24 (SecurityLab)
- pfSense OPT1: 192.168.60.1/24
- pfSense LAN: 192.168.50.1/24
- Web service: Nginx, TCP port 80
- Investigation tools: Wireshark, curl, pfSense firewall logs

## 3. Investigation Findings

### Finding 1 — Blocked TCP Connection Attempts

Classification: Expected — Authorized Security Testing

Kali initiated controlled TCP connection attempts to Ubuntu on destination ports 8080 and 8081. The connections timed out.

pfSense firewall logs confirmed that the default deny IPv4 rule blocked the SYN packets on OPT1.

Wireshark analysis of the original TCP 8080 attempt identified one initial SYN and four retransmissions using the same source port and raw TCP sequence number.

No successful TCP connection was observed on these destination ports during the tests.

### Evidence

- Screenshot 11: TCP 8080 SYN retransmissions
- Screenshot 12: pfSense TCP 8080 default deny logs
- Screenshot 13: pfSense TCP 8081 default deny logs
- Screenshot 15: TCP SYN retransmission analysis
- Screenshot 16: Raw pfSense firewall logs
- Packet capture: tcp8080-syn-retransmissions.pcapng

## 4. Preliminary Conclusion

The observed activity was expected and authorized. Repeated blocked SYN packets alone do not establish malicious intent. Correlating firewall logs with packet captures helped distinguish individual connection attempts from TCP retransmissions.

Further investigation and defensive validation will be documented in subsequent sections.

## 5. Defensive Response and Validation

A temporary, narrowly scoped firewall rule was created on pfSense OPT1 to explicitly block TCP traffic from Kali (192.168.60.100) to Ubuntu (192.168.50.100) on destination port 8080. Logging was enabled.

Testing confirmed that the custom rule blocked the connection attempts and generated identifiable firewall log entries. A separate HTTP test returned HTTP 200 OK on TCP port 80, demonstrating that authorized web traffic remained accessible.

After testing, the temporary rule was removed. A subsequent TCP 8080 connection attempt timed out, and firewall logs confirmed that the default deny IPv4 rule (1000000103) continued blocking the traffic.

### Evidence

- Screenshot 17: Explicit TCP 8080 Block rule
- Screenshot 18: Custom Block rule firewall logs
- Screenshot 19: HTTP 200 OK after firewall change
- Screenshot 20: Default deny rule restored

### Outcome

The defensive control was validated without disrupting authorized HTTP traffic. The temporary rule was removed, and the original default deny protection remained effective.

## 6. Investigation Timeline

All timestamps are recorded in UTC.

| Timestamp (UTC) | Event | Observation |
|---|---|---|
| Oct 30, 02:27:43 | Normal HTTP traffic | TCP port 80 allowed |
| Oct 30, 02:44:43–47 | TCP 8080 test | Default deny blocked SYN packets |
| Oct 30, 02:50:14–18 | TCP 8081 test | Default deny blocked SYN packets |
| Oct 30, 03:23:26–30 | TCP 8080 retest | Raw logs confirmed default deny |
| Oct 30, 03:48:39–43 | Custom Block rule test | Explicit rule blocked TCP 8080 |
| Oct 30, 03:54:59–55:03 | Firewall restoration test | Default deny protection verified |

### Timeline Analysis

The investigation began with authorized baseline traffic,
followed by controlled connection attempts to blocked ports.

Firewall logs and Wireshark captures were correlated to
distinguish individual connection attempts from retransmissions.

A temporary explicit Block rule was validated and removed.
The original default deny policy remained effective.

## 7. Investigation Methodology

### 7.1 Establish Normal Traffic

Verified connectivity between Kali Linux and Ubuntu through pfSense. Captured ICMP Echo Request/Reply traffic and successful HTTP connections to TCP port 80 using Wireshark. Correlated permitted traffic with pfSense firewall logs.

### 7.2 Generate Controlled Test Traffic

Used curl to initiate authorized TCP connection attempts to destination ports 8080 and 8081. Both connections timed out. Reviewed pfSense logs to confirm that the default deny policy blocked the traffic.

### 7.3 Analyze TCP Retransmissions

Filtered Wireshark traffic by source and destination IP addresses, TCP ports, and TCP stream. Examined SYN flags, source ports, and raw TCP sequence numbers.

For the original TCP 8080 test, identified one initial SYN and four retransmissions with the same source port and raw sequence number. Distinguished repeated packets from separate connection attempts.

### 7.4 Correlate Firewall Evidence

Compared packet captures with pfSense firewall logs, including timestamps, source and destination addresses, destination ports, TCP flags, and rule identifiers.

### 7.5 Validate Defensive Controls

Created a temporary, narrowly scoped TCP 8080 Block rule with logging enabled. Verified that the rule matched test traffic while authorized HTTP remained accessible. Removed the temporary rule and confirmed that the default deny policy continued blocking TCP 8080.

### 7.6 Investigation Limitations

This was an isolated, authorized homelab investigation. The evidence demonstrates firewall behavior and TCP connection attempts, but does not establish malicious intent, compromise, or a real-world security incident.

## 8. Final SOC Assessment

### Classification

Expected Activity — Authorized Security Testing

### Findings

1. pfSense successfully permitted authorized ICMP and HTTP traffic between the designated lab hosts.
2. TCP connection attempts to ports 8080 and 8081 were blocked by the default deny firewall policy.
3. Wireshark analysis confirmed that five SYN packets in the original TCP 8080 test represented one initial connection attempt and four retransmissions, not five independent attempts.
4. Firewall logs identified the source, destination, TCP flags, interface, and rule responsible for blocking traffic.
5. A temporary explicit Block rule successfully matched TCP 8080 traffic without disrupting authorized HTTP traffic on port 80.
6. After removing the temporary rule, the default deny policy continued blocking TCP 8080.

### Security Assessment

No successful TCP connection to ports 8080 or 8081 was observed during the controlled tests. The available evidence does not indicate a compromise or unauthorized intrusion.

### Lessons Learned

- Multiple firewall log entries do not necessarily represent multiple connection attempts.
- TCP retransmissions can be identified by correlating source ports, TCP flags, sequence numbers, and packet timing.
- Firewall rule order and specificity matter when implementing security controls.
- Packet captures and firewall logs provide complementary evidence during investigations.
- Security changes should be validated and temporary test configurations removed when no longer required.

## 9. Final Conclusion

This project demonstrated a structured SOC investigation workflow: establishing a baseline, generating controlled test traffic, analyzing packets, correlating firewall evidence, validating defensive controls, and restoring the original security configuration.

The investigation was completed in an isolated VirtualBox homelab. The findings represent authorized security testing rather than a real-world security incident.
