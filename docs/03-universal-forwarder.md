## Deployment Summary

Platform:
- Ubuntu 24.04 LTS

Role:
- Log Collection

Destination:
- Splunk Enterprise

Forwarding Protocol:
- TCP 9997

Current Data Sources:
- Fail2ban
- SSH Journald
- sudo Journald

Status:
- Operational
---

# Splunk Universal Forwarder

## Overview

Splunk Universal Forwarder is a lightweight agent used to collect log data from remote systems and securely forward it to a Splunk Enterprise instance.

In this lab, the Universal Forwarder runs on my Ubuntu server (`voidbox`) and forwards Linux logs to the Splunk Enterprise server Instance.

---

## Purpose

The Universal Forwarder was selected because it allows me to collect logs from the endpoint without installing a full instance of Splunk on the system.

Using the Universal Forwarder allows the Ubuntu server to:

- Monitor selected log sources
- Forward logs over TCP 9997
- Minimize resource usage
- Centralize analysis inside Splunk Enterprise

---

## Software

| Component | Version |
|-----------|---------|
| Splunk Universal Forwarder | 10.4.1 |
| Ubuntu | 24.04 LTS |

---

## Installation

The Universal Forwarder was downloaded as the Linux AMD64 tarball.

Installation location:

```text
/opt/splunkforwarder
```

Archive extraction:

```bash
tar -xzf splunkforwarder-10.4.1-5a009d941268-linux-amd64.tgz
sudo mv splunkforwarder /opt/
```

---

## Initial Configuration

The forwarder was configured to send data to the Splunk Enterprise server.

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.1.X:9997
```

Current forwarding status:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Expected result:

```text
Active forwards:
    192.168.1.X:9997
```

---

## Configured Inputs

The Universal Forwarder is currently configured to collect the following log sources:

- Fail2ban log
- SSH journal
- sudo journal

Example monitor:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
```

Journald inputs are configured using:

```text
/opt/splunkforwarder/etc/apps/journald_input/local/inputs.conf
```

---

## Validation

The following checks confirm that the Universal Forwarder is operating correctly.

Forwarder status:

```bash
sudo /opt/splunkforwarder/bin/splunk status
```

Forwarding status:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Splunk Search:

```spl
index=* host=voidbox
```
Successful search results confirm that the Universal Forwarder is forwarding data and that Splunk Enterprise is successfully indexing events from the Ubuntu server.
---

## Troubleshooting

During deployment the forwarder initially showed:

```text
Configured but inactive forwards
```

The issue was traced to:

- Windows Firewall
- Windows network profile
- Incorrect IP address during testing

Additional details are documented in:

[`troubleshooting.md`](troubleshooting.md).
---

## Lessons Learned

One of the biggest lessons from deploying the Universal Forwarder was that a configured destination does not necessarily indicate successful connectivity.

Splunk can successfully save a forwarding destination while still being unable to establish a network connection.

Validating firewall rules, routing, and listening ports proved just as important as configuring the forwarder itself.
