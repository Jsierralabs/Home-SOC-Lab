# Home SOC Lab

## Purpose

This Home SOC Lab was built to provide hands-on experience with security monitoring, log analysis, and system administration in a controlled environment.

While pursuing a career transition into cybersecurity, I wanted practical experience using the same technologies commonly found in Security Operations Centers (SOCs). Rather than relying solely on coursework or certifications, this project focuses on learning through implementation, troubleshooting, and documentation.

The objective is not simply to install security software, but to understand:

- How security data is generated
- How logs move between systems
- How a SIEM ingests and searches data
- How to investigate issues when something does not work as expected
- How to document technical work in a professional manner

---

## Goals

This lab is designed to help me develop practical experience with:

- Linux system administration
- Windows administration
- Splunk Enterprise
- Splunk Universal Forwarder
- Log forwarding
- Network troubleshooting
- Firewall configuration
- VPN technologies
- Detection engineering
- Security monitoring
- Documentation and change tracking

---

## Current Environment

### Windows Workstation

Role:

- Splunk Enterprise
- Search Head
- Indexer

### Ubuntu Server (voidbox)

Services:

- Splunk Universal Forwarder
- WireGuard VPN
- AdGuard Home
- Netdata
- SSH
- Fail2ban

---

## Current Project Status

Completed:

- Splunk Enterprise installed
- Developer License applied
- Universal Forwarder deployed
- Linux log forwarding operational
- SSH and sudo journal ingestion
- Fail2ban log ingestion
- Remote Splunk dashboard access through WireGuard
- Project documentation

In Progress:

- Splunk searches
- Dashboards
- Alert creation

Planned:

- Sysmon deployment
- Windows Event Log ingestion
- Detection rules
- Dashboards
- Additional Linux log sources

---

## Learning Philosophy

Every issue encountered during this project is documented.

Rather than removing mistakes, the repository keeps troubleshooting notes describing:

- Symptoms
- Root cause
- Investigation
- Resolution
- Lessons learned

The goal is to demonstrate not only the finished environment, but also the thought process used to diagnose and solve technical problems.
