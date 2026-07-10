# Splunk Enterprise

## Objective

Deploy a centralized SIEM for my Home SOC Lab using the Splunk Enterprise Developer License.

---

## Why Splunk?

Originally I planned to use Splunk Cloud Developer Edition to keep indexing workloads off my Ubuntu server.

During registration I learned the current Developer Program provides a 10 GB/day Splunk Enterprise Developer License instead of automatically provisioning a Splunk Cloud instance.

Rather than treating this as a setback, I redesigned the lab architecture.

---

## Environment

Host:
Windows 10 Desktop

Hardware

- Ryzen 5 3600
- 32 GB DDR4 RAM
- 1 TB SSD
- 2 TB HDD

---

## Installation

- Installed Splunk Enterprise
- Applied 10 GB Developer License
- Verified successful startup
- Verified Web UI access
- Confirmed license activation

---

## Architecture Decision

Instead of running Splunk on my Ubuntu server ("voidbox"), I chose to keep Splunk on my desktop.

Reasons:

- Desktop has significantly more CPU and RAM.
- Ubuntu server continues running infrastructure services:
  - AdGuard Home
  - WireGuard
  - Netdata
  - SSH

This keeps the infrastructure stable while allowing Splunk to perform indexing and searches on more capable hardware.

---

## Next Steps

- Enable receiving on TCP 9997
- Install Universal Forwarder on voidbox
- Forward Linux logs
- Install Sysmon
- Forward Windows Event Logs
- Build dashboards
