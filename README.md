# ls-rules

Firewall rules for SonarMD engineering machines — defense-in-depth with two independent enforcement layers.

## Architecture

```
+------------------------------------------+
|  Application Layer (LuLu)                |
|  -- Process identity (code signature)    |
|  -- Binary path validation               |
|  -- Per-app allow/block decisions        |
+------------------------------------------+
|  Network Layer (Little Snitch)           |
|  -- IP/CIDR rules                        |
|  -- Port restrictions                    |
|  -- Domain allowlists                    |
|  -- Protocol filtering                   |
+------------------------------------------+
```

Both firewalls enforce independently. Even if an attacker hijacks an allowed process, the other layer can catch it.

## Repository Structure

```
ls-rules/
  *.lsrules              # Little Snitch rule sets (root level)
  lulu/
    rules-work-hipaa.json    # LuLu profile: SonarMD dev/ops
    rules-lockdown.json      # LuLu profile: incident response
    rules-personal.json      # LuLu profile: personal / off-hours
    blocklist-domains.txt    # Domain blocklist (ad tracking, telemetry)
```

All files are auto-synced from `tonys-toolbox` on every push via pre-push hook.

---

## Little Snitch Rules

### Rule Files

| File | Description |
|------|-------------|
| [`sonarmd-hardened.lsrules`](./sonarmd-hardened.lsrules) | Hardened base rules for avespoli — browsers, dev tools, Microsoft 365, Docker, IT management, credential managers, telemetry blocking |

### Loading in Little Snitch

1. Open Little Snitch > Rules
2. Click **+** > **New Rule Group from URL**
3. Paste the raw URL:

```
https://raw.githubusercontent.com/sonarmd/ls-rules/main/sonarmd-hardened.lsrules
```

4. Set update interval: **Daily**

### What These Rules Cover

| App / Process | Policy |
|---|---|
| Chrome | Allow out 80/443 - Deny incoming |
| Terminal | Allow out any - Deny incoming |
| Microsoft Teams | Allow out 443 + UDP 3478-3481 (STUN/TURN) - Deny incoming |
| Microsoft Outlook | Allow out 443 - Deny incoming |
| Safari | Allow out 443 only - Deny incoming |
| Docker | Allow out 443 - Allow incoming from 127.0.0.1 only - Deny external incoming |
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

---

## LuLu Rules

LuLu is an app-level firewall from Objective-See. It handles process-level allow/block decisions and code signature validation — complementing Little Snitch's network-level enforcement.

### Rule Files

| File | Profile | Description |
|------|---------|-------------|
| [`lulu/rules-work-hipaa.json`](./lulu/rules-work-hipaa.json) | `work-hipaa` | SonarMD dev/ops — only approved tools get outbound access |
| [`lulu/rules-lockdown.json`](./lulu/rules-lockdown.json) | `lockdown` | Incident response / SOC2 audit — deny-all with surgical exceptions |
| [`lulu/rules-personal.json`](./lulu/rules-personal.json) | `personal` | Personal / off-hours — relaxed but still blocks telemetry |
| [`lulu/blocklist-domains.txt`](./lulu/blocklist-domains.txt) | All | Domain blocklist for ad tracking, telemetry, and fingerprinting |

### Loading in LuLu

1. Clone this repo: `git clone git@github.com:sonarmd/ls-rules.git`
2. Open LuLu menu bar > **Profiles** > **Add Profile**
3. Create profiles: `work-hipaa`, `lockdown`, `personal`
4. For each profile:
   - Activate it: LuLu menu bar > Profiles > [select profile]
   - Import rules: LuLu menu bar > Rules > Import > select the corresponding `lulu/rules-*.json`
5. Install blocklist:
   - LuLu > Settings > Lists > Block List > **+** > **Local**
   - Select `lulu/blocklist-domains.txt`

### Division of Responsibility

| Concern | Little Snitch | LuLu |
|---------|--------------|------|
| IP/CIDR restrictions | Primary | Not supported |
| Port-level rules | Primary | Not supported |
| Per-app allow/block | Can do | Primary (simpler UX) |
| Code signature validation | No | Yes (shows in alerts) |
| Domain-based blocklists | Yes | Yes (Lists feature) |
| Profile switching | No | Yes |

---

## Notes

- These rules complement the threat intel subscription groups (ThreatFox, URLHaus, PhishingArmy, HaGeZi Pro) — they are not a replacement.
- Factory/system rules (DHCP, mDNS, OCSP, trustd, etc.) are untouched and managed by Little Snitch natively.
- The `appGroup.macOS` deny-incoming rule must be set manually in the GUI — it uses an internal LS identifier not supported in remote rule files.
- LuLu profiles require v4.x+. Check version: LuLu menu bar > About.
- The Chrome/ksfetch binary replacement attack is real — LuLu's binary integrity monitoring helps detect it.
