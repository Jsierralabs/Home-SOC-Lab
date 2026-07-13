# Troubleshooting Notes

This document records issues encountered while building the Home SOC Lab, along with the troubleshooting steps, root causes, and resolutions.

The goal is to document the reasoning process behind each fix rather than only showing the final working configuration.

---

## Table of Contents

- [Splunk Universal Forwarder Configured but Inactive](#splunk-universal-forwarder-configured-but-inactive)
- [Accidental Broad Windows Firewall Rules](#accidental-broad-windows-firewall-rules)
- [Fail2ban Log File Not Found](#fail2ban-log-file-not-found)
- [Incorrect Splunk Forwarder Configuration Path](#incorrect-splunk-forwarder-configuration-path)
- [Fail2ban Four-Hour Timestamp Warning](#fail2ban-four-hour-timestamp-warning)
- [Splunk Search Failed Because of an Incomplete Comparator](#splunk-search-failed-because-of-an-incomplete-comparator)
- [Splunk Event Action Added an Exclusion Filter](#splunk-event-action-added-an-exclusion-filter)
- [Splunk Dashboard Using WireGuard VPN](#splunk-dashboard-using-wireguard-VPN)
- [Final Validation](#final-validation)

---

## Splunk Universal Forwarder Configured but Inactive

### Symptoms

The Splunk Universal Forwarder was successfully configured to send data to the Windows Splunk Enterprise server:

```text
192.168.1.X:9997
```

However, the forwarding status showed:

```text
Active forwards:
    None

Configured but inactive forwards:
    192.168.1.X:9997
```

This confirmed that the destination was saved, but the Universal Forwarder could not establish a TCP connection.

### Initial Checks

The Splunk Enterprise server had already been configured to receive data on TCP port `9997`.

A connection test was performed from the Ubuntu server using Bash's built-in TCP support:

```bash
timeout 5 bash -c '</dev/tcp/192.168.1.X/9997' \
  && echo CONNECTED || echo BLOCKED
```

The initial result appeared as:

```text
BLOCKED
```

### Root Causes

Two separate issues were identified.

#### 1. Incorrect IP Address During Testing

The first connectivity test mistakenly targeted:

```text
192.168.10
```

instead of:

```text
192.168.1.X
```

This caused the test to fail because the command was attempting to connect to the wrong network address.

#### 2. Windows Network Profile Was Set to Public

The Windows desktop was using the following network profile:

```text
NetworkCategory : Public
```

The Windows Firewall rule for Splunk was restricted to the `Private` profile, meaning it would not apply while the Ethernet connection was classified as Public.

### Resolution

The Windows Ethernet profile was changed to Private:

```powershell
Set-NetConnectionProfile -InterfaceIndex 4 -NetworkCategory Private
```

A tightly scoped Windows Firewall rule was created:

```powershell
New-NetFirewallRule `
  -DisplayName "Splunk Forwarder TCP 9997" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 9997 `
  -RemoteAddress 192.168.1.Y `
  -Action Allow `
  -Profile Private
```

The rule allows only:

- Inbound TCP traffic
- Destination port `9997`
- Traffic from the Ubuntu server at `192.168.1.Y`
- Traffic while the Windows network uses the Private profile

The connection test was then repeated using the correct IP address:

```bash
timeout 5 bash -c '</dev/tcp/192.168.1.X/9997' \
  && echo CONNECTED || echo BLOCKED
```

Result:

```text
CONNECTED
```

The Universal Forwarder was restarted and checked again:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Final result:

```text
Active forwards:
    192.168.1.X:9997
```

### Lesson Learned

A destination appearing under `Configured but inactive forwards` means the forwarding configuration exists, but Splunk cannot complete the network connection.

The troubleshooting process should verify:

1. Destination IP address
2. Destination port
3. Receiver listening state
4. Firewall scope
5. Active Windows network profile
6. Connectivity from the forwarding host

---

## Accidental Broad Windows Firewall Rules

### Symptoms

The following PowerShell command was entered without parameters:

```powershell
New-NetFirewallRule
```

PowerShell then prompted only for a display name.

This created an inbound firewall rule that was not restricted to TCP port `9997`, a source address, or a specific profile.

A second accidental rule was created when a PowerShell backtick was entered as part of the display name.

### Cause

In PowerShell, the backtick character:

```text
`
```

is used as a line-continuation character only when placed at the end of a command line.

It should not be entered into an interactive parameter prompt.

Running `New-NetFirewallRule` by itself caused PowerShell to enter interactive prompting mode instead of receiving the complete rule definition.

### Resolution

The accidental rules were removed using their unique rule names:

```powershell
Remove-NetFirewallRule -Name "{RULE-GUID}"
```

The correct rule was then created as a single command:

```powershell
New-NetFirewallRule -DisplayName "Splunk Forwarder TCP 9997" -Direction Inbound -Protocol TCP -LocalPort 9997 -RemoteAddress 192.168.1.Y -Action Allow -Profile Private
```

The completed rule was verified:

```powershell
Get-NetFirewallRule -DisplayName "Splunk Forwarder TCP 9997" |
Format-List DisplayName,Enabled,Profile,Direction,Action
```

Port configuration:

```powershell
Get-NetFirewallRule -DisplayName "Splunk Forwarder TCP 9997" |
Get-NetFirewallPortFilter
```

Address configuration:

```powershell
Get-NetFirewallRule -DisplayName "Splunk Forwarder TCP 9997" |
Get-NetFirewallAddressFilter
```

Expected values:

```text
Protocol      : TCP
LocalPort     : 9997
RemoteAddress : 192.168.1.Y
Profile       : Private
Direction     : Inbound
Action        : Allow
```

### Lesson Learned

When using PowerShell commands with several parameters, either:

- Paste the full command as one line, or
- Use a backtick only at the end of each continued line

Firewall rules should always be verified after creation to confirm their protocol, port, source address, direction, and profile scope.

---

## Fail2ban Log File Not Found

### Symptoms

The following monitor was initially attempted:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

Splunk returned:

```text
Parameter name: Path must be a file or directory.
```

### Cause

The Ubuntu server did not have `/var/log/auth.log`.

This host was using systemd journal for authentication events rather than a traditional authentication log file.

The server did contain:

```text
/var/log/journal
```

Fail2ban was also configured to use the systemd backend.

### Resolution

A file that existed on the system was used first to confirm the forwarding pipeline:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
```

The forwarder was restarted:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

The data was verified in Splunk with:

```spl
index=* host=voidbox source="/var/log/fail2ban.log"
```

After confirming that file-based forwarding worked, native journald inputs were configured for SSH and sudo events.

### Lesson Learned

Do not assume that every Ubuntu system has `/var/log/auth.log`.

Before configuring a file monitor, confirm that the file exists:

```bash
ls -l /var/log
```

For systems using systemd journal, use Splunk's journald input rather than attempting to monitor a nonexistent log file.

---

## Incorrect Splunk Forwarder Configuration Path

### Symptoms

While saving the journald input configuration in Nano, the following error appeared:

```text
Error writing /opt/splunk/forwarder/etc/apps/journald_input/local/inputs.conf:
No such file or directory
```

### Cause

The Splunk Universal Forwarder was installed at:

```text
/opt/splunkforwarder
```

The path was mistakenly entered as:

```text
/opt/splunk/forwarder
```

The extra slash caused Linux to interpret `splunk` and `forwarder` as separate directories.

### Resolution

The correct local configuration directory was created:

```bash
sudo mkdir -p /opt/splunkforwarder/etc/apps/journald_input/local
```

The correct file was opened:

```bash
sudo nano /opt/splunkforwarder/etc/apps/journald_input/local/inputs.conf
```

The file was saved using:

```text
Ctrl + O
Enter
Ctrl + X
```

The Universal Forwarder was then restarted:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

### Lesson Learned

Linux paths must be checked carefully because a single slash changes the directory structure.

The Universal Forwarder installation path used throughout this lab is:

```text
/opt/splunkforwarder
```

---

## Fail2ban Four-Hour Timestamp Warning

### Symptoms

Fail2ban produced warnings similar to:

```text
Detected a log entry 4h after the current time in operation mode.
This looks like a timezone problem.
Treating such entries as if they just happened.
```

An example event contained:

```text
2026-07-11T17:30:17.012825+00:00
```

The Fail2ban warning itself was logged at:

```text
2026-07-11 13:30:17
```

### Investigation

The system clock configuration was checked:

```bash
timedatectl
```

Results showed:

```text
Time zone: America/New_York
System clock synchronized: yes
NTP service: active
RTC in local TZ: no
```

The journal entry used UTC:

```text
2026-07-11T17:30:17+00:00
```

The Fail2ban log displayed local Eastern Daylight Time:

```text
2026-07-11 13:30:17
```

These timestamps represent the same moment:

```text
17:30 UTC = 13:30 EDT
```

The Fail2ban SSH jail was confirmed to be reading from systemd journal:

```bash
sudo fail2ban-client get sshd journalmatch
```

Result:

```text
_SYSTEMD_UNIT=sshd.service + _COMM=sshd
```

The jail was not monitoring a traditional file:

```bash
sudo fail2ban-client get sshd logpath
```

Result:

```text
No file is currently monitored
```

The installed version was also confirmed to be current:

```text
fail2ban 1.0.2-3ubuntu0.1
```

### Conclusion

The system time, timezone, NTP synchronization, and hardware clock configuration were correct.

The warning appears to be caused by Fail2ban interpreting a UTC journal timestamp as though it were local time.

Fail2ban continued processing the events and explicitly stated that the entries were being treated as current.

### Resolution

No system timezone changes were made.

The server remains configured with:

```text
America/New_York
```

The hardware clock remains in UTC.

The issue was documented as a timestamp parsing warning rather than a host clock problem.

### Lesson Learned

A timestamp warning does not automatically mean the system clock is incorrect.

When investigating time-related alerts, compare:

- Local time
- UTC time
- Timezone offset
- NTP state
- Event source format
- Parser behavior

Incorrect assumptions about timestamps can distort event correlation and incident timelines.

---

## Splunk Search Failed Because of an Incomplete Comparator

### Symptoms

The following search produced an error:

```spl
index=* host=voidbox sourcetype="linux:journald:sudo" COMMAND=
```

Splunk returned:

```text
Comparator '=' is missing a term on the right hand side.
```

### Cause

Splunk interpreted `COMMAND=` as a field comparison, but no value was provided after the equals sign.

Additionally, `COMMAND` was not yet an extracted field. It existed only inside the raw event text.

### Resolution

The raw event was searched and parsed using `rex`.

Working search:

```spl
index=* host=voidbox sourcetype="linux:journald:sudo"
| where like(_raw, "%COMMAND=%")
| rex field=_raw "COMMAND=(?<command>.*)"
| table _time command _raw
| sort - _time
```

A more detailed extraction was then created:

```spl
index=* host=voidbox sourcetype="linux:journald:sudo"
| rex field=_raw "^\s*(?<sudo_user>\S+)\s+:\s+TTY=(?<tty>[^;]+)\s*;\s*PWD=(?<pwd>[^;]+)\s*;\s*USER=(?<target_user>[^;]+)\s*;\s*COMMAND=(?<command>.*)$"
| where isnotnull(command)
| table _time sudo_user tty pwd target_user command
| sort - _time
```

This produced structured fields for:

```text
_time
sudo_user
tty
pwd
target_user
command
```

### Lesson Learned

If a value exists only inside `_raw`, it cannot be reliably searched as an extracted field until it is parsed.

The `rex` command can create fields at search time without changing the original indexed data.

---

## Splunk Event Action Added an Exclusion Filter

### Symptoms

After selecting an event and clicking an event action icon, Splunk generated:

```spl
_raw!="  lokzii : TTY=pts/2 ; PWD=/home/lokzii ; USER=root ; COMMAND=/usr/bin/date"
```

### Cause

The selected action excluded the event rather than adding it as an inclusion filter.

The operator:

```text
!=
```

means:

```text
does not equal
```

### Resolution

To include an exact event, use:

```spl
_raw="event text"
```

To search all sudo command events, use the structured extraction search instead of matching one exact raw event.

### Lesson Learned

Splunk event actions can either include or exclude selected values.

Always inspect the generated operator:

```text
=   include
!=  exclude
```
## Splunk Dashboard using WireGuard VPN

### Symptoms

After attempting to VPN into the LAN network we were able to see that the connection could not be completed. 

### Cause
```text
UFW was confirmed to be dropping forwarded traffic.

```bash
sudo iptables -S FORWARD

```text
-P FORWARD DROP
-A FORWARD -j ufw-before-logging-forward
-A FORWARD -j ufw-before-forward
-A FORWARD -j ufw-after-forward
-A FORWARD -j ufw-after-logging-forward
-A FORWARD -j ufw-reject-forward
-A FORWARD -j ufw-track-forward
```
```
---

## Final Validation

The final working log pipeline is:

```text
Ubuntu voidbox
    ↓
Splunk Universal Forwarder
    ↓
TCP 9997
    ↓
Windows Splunk Enterprise
    ↓
Indexed Fail2ban and journald events
```

The following data sources were successfully validated:

- Fail2ban logs
- SSH journal events
- sudo journal events
- sudo command field extraction

The lab now provides visibility into:

- SSH sessions
- sudo command execution
- root session creation
- Fail2ban jail startup and shutdown
- Fail2ban ticket flushing
- service restart behavior
- timestamp parsing warnings
- Universal Forwarder connectivity
