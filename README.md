<div align="center">

```
███████╗███████╗███╗   ██╗████████╗██╗███╗   ██╗███████╗██╗
██╔════╝██╔════╝████╗  ██║╚══██╔══╝██║████╗  ██║██╔════╝██║
███████╗█████╗  ██╔██╗ ██║   ██║   ██║██╔██╗ ██║█████╗  ██║
╚════██║██╔══╝  ██║╚██╗██║   ██║   ██║██║╚██╗██║██╔══╝  ██║
███████║███████╗██║ ╚████║   ██║   ██║██║ ╚████║███████╗███████╗
╚══════╝╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚═╝╚═╝  ╚═══╝╚══════╝╚══════╝
        F I R E W A L L   
```

**Linux-native · AI-driven · XDR · Deception · Zero Trust**

![Version](https://img.shields.io/badge/version-9.13-blue?style=for-the-badge)
![Language](https://img.shields.io/badge/C%2B%2B-17-00599C?style=for-the-badge&logo=c%2B%2B)
![Qt](https://img.shields.io/badge/Qt-6-41CD52?style=for-the-badge&logo=qt)
![License](https://img.shields.io/badge/license-Proprietary-red?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Engines](https://img.shields.io/badge/engines-62-purple?style=for-the-badge)
![Lines](https://img.shields.io/badge/lines-~35k-orange?style=for-the-badge)

</div>

---

## What is SentinelFirewall?

SentinelFirewall is a **Linux-native, headless security platform** written in modern C++17 with Qt 6. It is not a traditional firewall. It combines capabilities that most security products keep in separate tools — all compiled into a single binary, managed by a single daemon, correlated by a single AI council.

> Built for government and industrial threat models — energy, financial sector, and government enterprise environments.

---

## ⚡ Capability Overview

<table>
<tr>
<td width="50%">

**Detection**
- 🔍 Suricata-format IDS (TCP/UDP/HTTP/DNS/TLS/SMB)
- 📡 C2 beacon detection via FFT autocorrelation
- 🧬 Polymorphic & AI-generated malware classifier
- 🖥️ UEBA behavioral baselines + anomaly scoring
- 🔐 JA4/JA3 TLS fingerprinting (Cobalt Strike, Sliver, Meterpreter)
- 🏭 OT protocol parser (Modbus, S7Comm, BACnet, DNP3)

</td>
<td width="50%">

**Response**
- 🚫 6-mode device isolation via nftables + tc
- 🤖 8-agent AI council with autonomous patching
- 🪤 Active deception — 17 honeytoken types + tripwire
- 🔬 Capstone disassembly + Unicorn CPU emulation sandbox
- ☁️ Bidirectional AWS / Azure / GCP alert forwarding
- 📊 STIX 2.1 / OpenIOC / Plaso / CEF forensic export

</td>
</tr>
</table>

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│               Web Dashboard  :8444                       │
│         (Flask — index.html + dashboard_api.py)          │
└────────────────────────┬────────────────────────────────┘
                         │  REST API :8443
┌────────────────────────▼────────────────────────────────┐
│                    APIServer (413 LOC)                   │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│           FirewallEngine (1610 LOC) — orchestrator       │
│   Loads · Wires · Starts · Stops all 62 engines          │
└────────┬──────────────┬──────────────┬──────────────────┘
         │              │              │
    ┌────▼────┐   ┌─────▼─────┐  ┌────▼──────┐
    │   AI    │   │ Detection  │  │ Response  │
    │ Council │   │   Layer    │  │   Layer   │
    └────┬────┘   └─────┬─────┘  └────┬──────┘
         │              │              │
    8 LLM agents   IDS · EDR · XDR   AutoIsolation
    + MLTrainer    Beacon · ZTNA      CommanderAutoFix
    + OllamaBridge Container · DLP    SOAREngine
                         │
         ┌───────────────▼───────────────┐
         │   Forensic + Audit + Cloud    │
         │ ForensicTimeline · PostExploit │
         │ CloudConnector · SIEMConnector │
         └───────────────────────────────┘
```

---

## 🤖 The Eight AI Agents

The platform runs an **8-agent LLM council** via `AIOrchestrator` (1,276 LOC). All agents cooperate through a shared `KnowledgeBase`. Decisions are applied by `CommanderAutoFix`.

| # | Agent | Role | Cadence |
|---|-------|------|---------|
| 1 | **Commander** (Claude Opus 4) | Decision authority — approves or refuses all patches | Every 15 min |
| 2 | **Vulnerability Specialist** | NVD · CISA KEV · CVSS correlation → prioritized mitigations | On new CVE |
| 3 | **Anomaly Specialist** | UEBA baselines + beacon verdicts → insider threat detection | Continuous |
| 4 | **Malware Specialist** | Disassembly + YARA + packing analysis | On sample |
| 5 | **Triage Specialist** | XDR incident → one-paragraph analyst summary | On incident |
| 6 | **Red Team Specialist** | Finds under-defended attack paths | Hourly |
| 7 | **PenTester Specialist** | Live offensive tests against own perimeter | Scheduled |
| 8 | **Monitor Specialist** | Engine health + rate anomaly remediation | Every 30 s |

> A ninth **commander-autofix** Ollama student model keeps the platform autonomous when the Anthropic API is unreachable.

---

## 🪤 DeceptionEngine — v9.13 Highlight

**1,460 lines · STRONG tier** — Active deception layer hardened in this release.

### 17 Honeytoken Types

| Type | Generated Content |
|------|------------------|
| `CREDENTIAL` | Postgres-style DB connection string (user/pass/host/port/db) |
| `API_KEY` | Service-correct prefix — `sk_live_` Stripe, `sk-` OpenAI, `SG.` SendGrid |
| `SSH_KEY` | Full OpenSSH PEM — 24 lines of random base64 body |
| `AWS_KEY` | `AKIA...` access key + multi-profile credentials file with real region values |
| `GIT_TOKEN` | `ghp_` GitHub PAT in `.netrc` format |
| `SLACK_WEBHOOK` | Full `hooks.slack.com/services/T.../B.../...` URL |
| `JWT` | Real HS256 JWT with admin claims + 1-year expiry |
| `K8S_SA_TOKEN` | 192-char Kubernetes service account token |
| `DOCKER_TOKEN` | `dckr_pat_` in `config.json` with base64 auth |
| `JIRA_TOKEN` | `ATATT...` Atlassian API token in ini config |
| `DATABASE_URL` | postgres / mysql / mongodb / redis connection strings |
| `DNS_CANARY` | Unique `subdomain.canary.sentinel.local` FQDN |
| `WEB_BUG` | Tracking URL `canary.sentinel.local/trap/...` |
| `HONEYFILE` | Arbitrary decoy content with SHA-256 fingerprint |
| `BREADCRUMB` | Fake admin console URL + default creds bait |
| `FAKE_ADMIN_SHARE` | Directory with `admin_only.txt` + `backup_keys.txt` |
| `FAKE_DOCUMENT` | Markdown with embedded web bug tracking pixel |

### Tripwire Detection

| Mechanism | Detects |
|-----------|---------|
| `QFileSystemWatcher` | Any write/modify to a decoy path — fires immediately |
| Atime polling (30 s) | Reads — `stat()` each decoy, compare atime. Advance = unauthorized read |
| DNS canary | DNSFirewall hook → `reportDNSQuery()` on any `*.canary.sentinel.local` query |
| Web bug | SSLInspector/WAF hook → `reportWebHit()` on HTTP request to tracking URL |

### On Every Trigger — Three-Way Alert Forwarding

```
Token touched
     │
     ├──▶ PostExploitLogger  →  CRITICAL · MITRE T1552 + T1083 · attack chain · fix
     ├──▶ XDREngine          →  XDREvent  phase=CREDENTIAL_ACCESS
     └──▶ ForensicTimeline   →  Audit-grade record · full metadata
```

> **Token rotation:** `rotateExpiredTokens()` replaces untriggered tokens older than 7 days. Catches attackers who exfiltrated a credential but haven't used it yet.

> **Default plant:** `/var/lib/sentinel/decoys/` auto-populated on first start with 16 file decoys + 2 DNS canaries + 1 web bug. Survives restart via JSONL persistence.

---

## 📋 Engine Catalog

**62 engines · ~35,000 lines**

<details>
<summary><b>🔵 STRONG tier — 25 engines (500+ lines each)</b></summary>

| Engine | LOC | What it does |
|--------|-----|--------------|
| `FirewallEngine` | 1610 | Root orchestrator — owns, wires, and starts all 62 engines |
| `AIOrchestrator` | 1276 | 8-agent LLM council — prompts, cadence, Commander authority |
| `CloudConnector` | 1277 | AWS SigV4 + Azure OAuth2 + GCP RS256 JWT — bidirectional cloud SIEM |
| `ZTNABroker` | 1224 | Zero Trust broker — short-lived JWTs, posture checks, MFA escalation |
| `ContainerSecurity` | 1122 | Docker/K8s runtime monitoring — Trivy CVE scan, drift detection, auto-pause |
| `PostExploitLogger` | 1024 | Vulnerability ledger — SHA-256 audit chain, STIX/OpenIOC/CSV export |
| `HoneypotEngine` | 1042 | 11-protocol honeypot — SSH, Telnet, FTP, HTTP, SMB, RDP, MSSQL, MySQL, Postgres, Modbus |
| `PenTesterAgent` | 1128 | Offensive self-test — Atomic Red Team scenarios, confirms each is caught |
| `MLTrainer` | 1060 | sklearn pipeline — IsolationForest, RandomForest k-fold, PSI drift, 9 Ollama student models |
| `DeceptionEngine` | 1460 | 17 honeytoken types, 4 tripwire mechanisms, 3-way alert forwarding |
| `ForensicTimeline` | 1105 | 50k-event log — Plaso import, STIX 2.1, OpenIOC, CEF, graph reconstruction |
| `DeviceFingerprint` | 1573 | 8-channel device ID — MAC OUI, p0f TCP, JA4, mDNS, SNMP, ports, hostname |
| `DeviceController` | 1017 | 6 isolation modes — nftables + tc + SSH + Wake-on-LAN |
| `BeaconDetector` | 944 | FFT autocorrelation C2 beaconing — DGA entropy, DNS tunneling |
| `XDREngine` | 904 | Cross-engine correlation graph — kill-chain incident assembly |
| `IDAEngine` | 889 | Capstone disassembly + Unicorn emulation — CFG, packer detection, suspicious API chains |
| `DownloadInspector` | 888 | YARA + sandbox + quarantine on malicious downloads |
| `RedTeamHunter` | 888 | Finds under-defended attack paths from CVE feed + asset config |
| `DeviceScanner` | 939 | ARP + mDNS + SNMP + NetBIOS + active probing device discovery |
| `WindowsDeviceAgent` | 821 | WMI + PowerShell Remoting — pull/push remote Windows endpoint state |
| `AIPentestDefense` | 765 | Detects AI-assisted attackers — rapid CVE chains, LLM-formatted payloads |
| `VulnerabilityFeedEngine` | 620 | NVD + CISA KEV + Vulners + OSV feed — CVSS, EPSS, known-exploited flag |
| `PolymorphicAnalyzer` | 616 | Classifies polymorphic/metamorphic/packed/fileless/BYOVD/AI-generated malware |
| `RuleEngine` | 533 | Policy YAML → predicates → cross-engine "if X then Y" enforcement |
| `CheckpointEngine` | 663 | Signed state snapshots for disaster recovery + compliance audit |
| `JA4Fingerprinter` | 537 | JA4/JA4T/JA4S/JA4H — identifies Cobalt Strike, Sliver, Mythic, Metasploit |
| `SSLInspector` | 592 | Optional TLS MITM — decrypted payloads flow to DLP, WAF, YARA |
| `SandboxDetonator` | 526 | Firejail/nspawn detonation — filesystem diff, dropped files, process tree |
| `PythonBridge` | 502 | JSON stdin/stdout Python subprocess shim for sklearn, Suricata, MISP |
| `AICodeContextAnalyzer` | 573 | LLM explanation of disassembled binary control flow |
| `EncryptedTrafficAnalyst` | 499 | JA4 + cert chain + SNI + ratio analysis — C2-over-TLS without decryption |
| `VPNTorDetector` | 495 | IP reputation + TLS fingerprint — VPN provider and Tor exit node detection |
| `IDSEngine` | 788 | Suricata-format IDS — TCP/UDP/ICMP/HTTP/DNS/TLS/SMB/FTP decoders |

</details>

<details>
<summary><b>🟢 GOOD tier — 20 engines (300–499 lines)</b></summary>

| Engine | LOC | What it does |
|--------|-----|--------------|
| `KnowledgeBase` | 428 | Mutex-protected shared AI memory with TTL |
| `HAController` | 421 | Primary/standby HA — UDP heartbeat, state snapshot, auto-promotion |
| `CommanderAutoFix` | 418 | Autonomous patcher — nft rules, sysctl, config, systemctl with diff log |
| `NACController` | 416 | 802.1X + RADIUS shim network access control |
| `SOAREngine` | 413 | 14 YAML playbooks — isolate, query TI, Slack, page on-call |
| `APIServer` | 413 | JWT-protected HTTP/HTTPS REST API on :8443 |
| `AntiVirus` | 413 | ClamAV + YARA fast-path filter before heavy analysis |
| `AlertManager` | 390 | Dedup + severity routing → syslog / email / webhook / SIEM |
| `EDREngine` | 389 | Endpoint coordinator — /proc snapshots, FS changes, syscall traces |
| `EBPFLoader` | 377 | XDP kernel program loader — line-rate filtering, per-CPU hash maps |
| `AutoIsolation` | 378 | Isolation orchestration — decides scope (host/VLAN/subnet) |
| `MemoryForensics` | 356 | Live /proc/PID/mem analysis — unbacked memory, DLL injection, hollowing |
| `OllamaBridge` | 347 | HTTP wrapper for local Ollama LLM — offline fallback + student model fine-tuning |
| `SigmaHunter` | 340 | Sigma-format rule scanner against ForensicTimeline events |
| `SignatureUpdater` | 330 | Atomic signature updates from YARA, Suricata-rules, MISP, Sigma |
| `HealthMonitor` | 384 | Polls all engines every 30 s — stuck, leaking, rate anomaly detection |
| `UEBAEngine` | 462 | Per-entity behavioral baselines + anomaly scoring + ML training feed |
| `OTProtocolParser` | 475 | Modbus, S7Comm, EtherNet/IP, BACnet, DNP3, IEC 60870-5-104 |
| `DLPEngine` | 436 | PAN/Luhn, SSN, IBAN, passport — content inspection with block/quarantine |
| `SecretStore` | 300 | AES-256 at-rest secret storage (master.key 0600) |

</details>

<details>
<summary><b>🟡 OK tier — 14 engines (200–299 lines)</b></summary>

| Engine | LOC | What it does |
|--------|-----|--------------|
| `WAFEngine` | 290 | OWASP CRS — SQLi, XSS, path traversal, command injection, ML anomaly mode |
| `GeoBlocker` | 291 | MaxMind GeoLite2 — country-level block for inbound/outbound |
| `DNSFirewall` | 285 | Sinkhole DNS filter — blocklist + ForensicTimeline logging |
| `EmailDLP` | 274 | SMTP-out Postfix milter — attachment blacklist + encrypted archive detection |
| `SIEMConnector` | 312 | Splunk HEC / Elastic / Sentinel / QRadar / ArcSight CEF outbound bridge |
| `YARARuleManager` | 299 | YARA rule DB — compiled bytecode, hot-reload, scan(buffer/file) |
| `DarkwebMonitor` | 228 | Paste sites + Tor forums + breach DBs — domain/IP/email mention alerts |
| `DepartmentManager` | 208 | Multi-tenancy — Finance / OT / GuestWiFi rule and alert scoping |

</details>

<details>
<summary><b>🔴 THIN tier — 2 engines (next to be hardened)</b></summary>

| Engine | LOC | Status |
|--------|-----|--------|
| `DepartmentManager` | 208 | Cross-department audit policies needed |
| `D3FENDMapper` | 182 | MITRE D3FEND coverage queries needed |

</details>

---

## 📊 Engine Maturity at a Glance

```
STRONG  ████████████████████████░  28 engines  45%
GOOD    ████████████████████░░░░░  20 engines  32%
OK      ██████████████░░░░░░░░░░░  14 engines  23%
THIN    ░░░░░░░░░░░░░░░░░░░░░░░   0 engines   0%
STUB    ░░░░░░░░░░░░░░░░░░░░░░░░░   0 engines   0%
```

---

## 🚀 Build & Install

### Dependencies

```bash
# System packages
apt install build-essential cmake \
  qt6-base-dev qt6-base-private-dev libqt6sql6-sqlite \
  libcap-dev libnetfilter-conntrack-dev libnetfilter-queue-dev \
  libpcap-dev libnfnetlink-dev libssl-dev \
  libcapstone-dev libunicorn-dev libbpf-dev \
  clang llvm bpftool \
  yara libyara-dev clamav clamav-daemon \
  nftables tc wakeonlan etherwake \
  suricata zeek

# Python packages
pip3 install --break-system-packages \
  scikit-learn numpy pandas flask requests \
  boto3 azure-identity azure-mgmt-security \
  google-cloud-securitycenter
```

### Build & Start

```bash
cd ~/Desktop/SentinelV3
sudo bash install.sh

sudo systemctl enable --now sentinel
sudo journalctl -fu sentinel
```

> `install.sh` builds the C++ project, installs the systemd unit, drops config templates into `/etc/sentinel/`, and creates `/var/lib/sentinel/` and `/var/log/sentinel/`.

---

## ⚙️ Configuration

| File | Purpose |
|------|---------|
| `/etc/sentinel/sentinel.json` | Master config — which engines start, poll intervals, interface bindings |
| `/etc/sentinel/secrets.json` | API keys, cloud credentials, JWT signing keys (AES-256 encrypted, mode 0600) |
| `/etc/sentinel/master.key` | 32-byte random master key — generated once by install.sh. **Don't lose this.** |
| `/etc/sentinel/rules/*.yaml` | RuleEngine policy YAML — live-reloaded |
| `/etc/sentinel/yara/` | YARA rules — hot-reloaded by YARARuleManager |
| `/etc/sentinel/suricata-rules/` | Suricata rules — loaded by IDSEngine |

---

## 🛠️ Common Operations

```bash
# Live log
sudo journalctl -fu sentinel

# Block a country
# Edit /etc/sentinel/geoblock.json → restart GeoBlocker via dashboard

# Add a YARA rule
# Drop file into /etc/sentinel/yara/ → SignatureUpdater hot-reloads

# Export forensic timeline to STIX 2.1
# Dashboard → Timeline → Export STIX 2.1
# Output → /var/lib/sentinel/exports/

# Verify audit chain integrity
# Dashboard → Findings → Verify Audit Chain
# (also runs automatically on every daemon restart)
```

---

## 🗺️ Roadmap

| Version | Target | Status |
|---------|--------|--------|
| **v9.13** | DeceptionEngine — 17 honeytokens, tripwire, 3-way forwarding | ✅ Done |
| v9.14 | DepartmentManager hardening | ✅ Done |
| v9.15 | DarkwebMonitor hardening | ✅ Done |
| v9.16 | D3FENDMapper hardening | ✅ Done |

After **v9.16**: all 62 engines at GOOD or STRONG — no THIN, no STUB. ~36,000 lines.

Post-v9.16 focus shifts to:
- Performance profiling under realistic packet loads
- Operator documentation
- Real test environment deployment
- Compliance work

---

## 📦 Supporting Components

| Component | Purpose |
|-----------|---------|
| `ebpf/xdp_firewall.c` | Kernel XDP filter — line-rate packet filtering, per-CPU hash maps |
| `sentinel_ml_helper.py` | sklearn subprocess — IsolationForest, RandomForest, DBSCAN, PSI drift |
| `python/suricata_integration.py` | Feeds Suricata `eve.json` events into IDSEngine |
| `python/train_eta_model.py` | Offline EncryptedTrafficAnalyst training → ONNX export |
| `dashboard/index.html` + `dashboard_api.py` | Flask dashboard on `:8444` |

---

## 📜 License & Attribution

© 2026 Manaf AL-Dulaimi. All rights reserved.

This codebase incorporates the following open-source dependencies under their respective licenses:

| Dependency | License |
|------------|---------|
| Qt 6 | LGPL v3 |
| Capstone | BSD-3 |
| Unicorn Engine | GPLv2 |
| LIEF | Apache 2.0 |
| libbpf | LGPL v2.1 / BSD-2 |
| OpenSSL | Apache 2.0 |
| libpcap | BSD-3 |
| nftables / iproute2 | GPL |
| Suricata | GPLv2 |
| YARA | BSD-3 |
| ClamAV | GPLv2 |

> Trademarks referenced (AWS, Azure, GCP, Mandiant, Anthropic, Claude, Ollama, Kali, etc.) are the property of their respective owners and are used for technical interoperability purposes only.

---

<div align="center">

**Built with discipline — one engine per session, real algorithms, build stable.**

![GitHub last commit](https://img.shields.io/github/last-commit/Manaf-DEV1/Sentinel-Firewall?style=flat-square)
![GitHub repo size](https://img.shields.io/github/repo-size/Manaf-DEV1/Sentinel-Firewall?style=flat-square)

</div>
