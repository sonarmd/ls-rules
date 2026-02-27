# ls-rules

Hardened Little Snitch rule sets for SonarMD engineering machines.

## Rule Files

| File | Description |
|------|-------------|
| [`sonarmd-hardened.lsrules`](./sonarmd-hardened.lsrules) | Hardened base rules for avespoli — browsers, dev tools, Microsoft 365, Docker, IT management, credential managers, telemetry blocking |

## Loading in Little Snitch

1. Open Little Snitch → Rules
2. Click **+** → **New Rule Group from URL**
3. Paste the raw URL:

```
https://raw.githubusercontent.com/sonarmd/ls-rules/main/sonarmd-hardened.lsrules
```

4. Set update interval: **Daily**

## What These Rules Cover

| App / Process | Policy |
|---|---|
| Chrome | Allow out 80/443 · Deny incoming |
| Terminal | Allow out any · Deny incoming |
| Microsoft Teams | Allow out 443 + UDP 3478–3481 (STUN/TURN) · Deny incoming |
| Microsoft Outlook | Allow out 443 · Deny incoming |
| Safari | Allow out 443 only · Deny incoming |
| Docker | Allow out 443 · Allow incoming from 127.0.0.1 only · Deny external incoming |
| 1Password | Allow out 443 |
| Centrastage (Datto RMM) | Allow out 443 |
| Claude Desktop | Allow out 443 |
| Keeper Security | Allow out 443 |
| Microsoft Word | Allow out 443 |
| Little Snitch NE | Allow out any (required for LS daemon) |
| dnscrypt-proxy | Allow out 443 (DoH) + 853 (DoT) |
| WebStorm | Deny all outgoing |
| Sublime Text | Deny all outgoing |
| Microsoft AutoUpdate | Deny all outgoing |
| Apple Promoted Content | Deny all outgoing |
| Apple Weather Widget | Deny all outgoing |

## Notes

- These rules complement the threat intel subscription groups (ThreatFox, URLHaus, PhishingArmy, HaGeZi Pro) — they are not a replacement.
- Factory/system rules (DHCP, mDNS, OCSP, trustd, etc.) are untouched and managed by Little Snitch natively.
- The `appGroup.macOS` deny-incoming rule must be set manually in the GUI — it uses an internal LS identifier not supported in remote rule files.
