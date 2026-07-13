                    Internet
                        │
             Remote SSH / VPN
                        │
                ┌──────────────┐
                │   voidbox    │
                │ Ubuntu 24.04 │
                ├──────────────┤
                │ AdGuard      │
                │ WireGuard    │
                │ Netdata      │
                │ SSH          │
                │ UF           │
                └──────┬───────┘
                       │
             Splunk Forwarding
                 TCP 9997
                       │
                ┌──────▼───────┐
                │ Windows 10   │
                │ Splunk Ent.  │
                │ Search Head  │
                │ Indexer      │
                └──────┬───────┘
                       │
                 Windows Logs
                 Sysmon
                 Defender
