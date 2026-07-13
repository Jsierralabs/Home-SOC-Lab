```text
                    Internet
                        │
              SSH / WireGuard VPN
                        │
                ┌──────────────┐
                │   voidbox    │
                │ Ubuntu 24.04 │
                ├──────────────┤
                │ AdGuard Home │
                │ WireGuard    │
                │ Netdata      │
                │ SSH          │
                │ Splunk UF    │
                │ Fail2ban     │
                │ Journald     │
                │ sudo / SSH   │
                └──────┬───────┘
                       │
             Splunk Forwarding
                 TCP 9997
                       │
                ┌──────▼───────┐
                │ Windows 10   │
                │ Splunk Ent.  │
                ├──────────────┤
                │ Search Head  │
                │ Indexer      │
                │ Windows Logs │
                │ Sysmon       │
                │ Defender     │
                └──────────────┘
```
