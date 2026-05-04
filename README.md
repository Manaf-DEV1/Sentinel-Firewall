# 🛡 SentinelFirewall v5

> **AI-Orchestrated Next-Generation Firewall Platform for Government & Enterprise Production Deployment**

Developed independently by **Manaf AL-Dulaimi** · Copyright 2026 · All Rights Reserved

---

## 📊 At a Glance

| Metric | Value |
|--------|-------|
| Engine Subsystems | **42** |
| AI Agents (24/7) | **6** |
| JA4+ Fingerprints | **308,683** |
| Red-Team Cycle | **Every 15 minutes** |
| KEV Auto-Patch Speed | **< 15 minutes** |
| Annual Cost | **~$12,000/yr** |
| Full Install Time | **5 minutes** |
| Health Check Points | **9/9** |

---

## 🧠 AI Commander Architecture

SentinelFirewall v5 runs a **hierarchical multi-agent AI system** powered by Anthropic's Claude models, with an optional fully sovereign on-premise AI layer.

```
┌─────────────────────────────────────────┐
│         AI COMMANDER (Opus 4.7)         │
│   Coordinates all agents · High-level   │
│              decisions                  │
└──────────────────┬──────────────────────┘
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│Vuln Agent│ │Anomaly   │ │Malware   │ │Triage    │ │Monitor   │
│Sonnet 4.6│ │Sonnet 4.6│ │Sonnet 4.6│ │Sonnet 4.6│ │Haiku 4.5 │
└──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
                   │
     ┌─────────────┴──────────────┐
     ▼                            ▼
┌──────────────────┐   ┌─────────────────────┐
│  PenTesterAgent  │   │    RedTeamHunter     │
│ Opus 4.7·15 min  │   │  Opus 4.7 · 1 hour  │
└──────────────────┘   └─────────────────────┘
                   │
     ┌─────────────▼──────────────┐
     │    Ollama 70B (On-Prem)    │
     │  Sovereign AI · 40GB Model │
     │  Weekly MLTrainer fine-tune│
     └────────────────────────────┘
```

All agent findings are logged to `ollama_teaching.jsonl` and used to continuously train the local 70B model. Over time, the local LLM progressively replaces cloud API calls — achieving **full sovereign AI operation with zero foreign cloud dependency**. This is the only firewall platform in the world with this capability.

---

## 🔍 Detection & Prevention Engines

### 🔐 JA4+ TLS Fingerprinting
Identifies **308,683 known software fingerprints** from live network traffic — including all known Command & Control (C2) frameworks used by threat actors. Stored in `ja4_db.sqlite`. Goes far beyond traditional IP or port-based blocking.

**Tags:** `TLS Fingerprinting` `C2 Detection` `308K Signatures`

### 🛡 Suricata IDS
Full Suricata Intrusion Detection System runs as a dedicated process, reading EVE JSON logs in real-time. Detects intrusion patterns, protocol anomalies, and signature-based attacks across all network traffic.

**Tags:** `Real-time IDS` `EVE JSON` `Signature-based`

### ⚙️ nftables Kernel Firewall
Linux kernel-level packet filtering with **47 active rule lines** loaded at boot. Handles stateful packet inspection, connection tracking, NAT, and port-based filtering at the kernel level — before traffic reaches any application.

**Tags:** `47 Rules` `Kernel-level` `Stateful`

### ⚡ eBPF / XDP Kernel Programs
Extended Berkeley Packet Filter programs compiled into the Linux kernel for in-kernel traffic processing at line speed. XDP (eXpress Data Path) intercepts packets before the network stack — the fastest possible filtering layer.

**Tags:** `eBPF` `XDP` `Line-speed`

### 🚫 Rate-Limiting & Auto-Ban
**3-strike rule** on all API endpoints — after 3 failed authentication attempts, the source IP is automatically banned. Prevents brute force, credential stuffing, and denial-of-service attempts.

**Tags:** `3-Strike Rule` `Auto-Ban` `DoS Protection`

### 🌍 Geo-Blocking
Country-level traffic filtering configurable directly from the dashboard. Block entire nations or regions at the network level. Customer-specific blocked country lists can be set per deployment.

**Tags:** `Country Blocking` `Dashboard Control` `Per-deployment`

---

## 🕸 Deception Engine & Honeypots

SentinelFirewall v5 goes beyond detection — it actively deceives and fingerprints attackers.

| Honeypot | Port | Purpose |
|----------|------|---------|
| SSH Honeypot | `2222` | Captures SSH scanning, credential attempts, and attacker fingerprints |
| HTTP Honeypot | `8080` | Captures web exploits, CVE probes, scanner fingerprints, and web shell attempts |
| FTP Honeypot | `2121` | Detects legacy protocol probing — a strong indicator of automated threat toolkits |

**DeceptionEngine** — Beyond static honeypots, the DeceptionEngine plants **customer-specific honeytokens** in relevant locations within the customer's environment. Any access to these tokens immediately signals a breach or insider threat. Honeytokens are customized per deployment during production rollout.

---

## ⚡ AI Automation & Self-Healing

### 🔄 CISA KEV Auto-Apply
Monitors the CISA Known Exploited Vulnerabilities catalog continuously. When a new critical CVE is listed, Sentinel automatically applies the patch or mitigation **within 15 minutes** of publication — before most security teams have even read the alert.

Default threshold: `CVSS ≥ 9.8`

### 🔴 PenTesterAgent
A dedicated Claude Opus 4.7 instance runs as a **continuous penetration tester** — probing the defended environment for vulnerabilities every **15 minutes**. Discovers novel attack vectors before adversaries do. All findings teach the Ollama 70B model.

### 🕵️ RedTeamHunter
Runs a comprehensive self-audit **every hour**. Simulates advanced persistent threat (APT) attack patterns against the protected network. Every finding is logged to `ollama_teaching.jsonl` — training the local AI to recognize and counter the same patterns.

### 🧬 MLTrainer
Fine-tunes the local Ollama 70B model on a **weekly schedule** using all accumulated teaching data. Over time, the model develops specialization in the specific threat landscape of the deployed environment — truly bespoke AI security intelligence.

---

## 🔒 Security Hardening & Compliance

### 🖥 Kernel Hardening
**32 kernel sysctls** tuned for maximum security hardening:

`ASLR = 2` `SYN Flood Protection` `IP Spoofing Defense` `ICMP Redirect Block` `Core Dump Disable` `32 Total Sysctls`

### 🛡 AppArmor Sandboxing
Mandatory Access Control via AppArmor profiles on all processes. Restricts what files, network sockets, and capabilities each process can access — even if an attacker achieves code execution, AppArmor limits lateral movement and damage.

### 📋 auditd Audit Trail
Complete system-level audit logging via Linux `auditd`. Records all system calls, file access, authentication events, and privilege escalations. Required for SIEM integration and compliance frameworks: **NIS2, DORA, BSI-Grundschutz, ISO 27001**.

### 🔑 fail2ban
Automated threat response for repeated authentication failures across all services. Monitors log files in real-time and dynamically updates firewall rules to ban IPs exceeding threshold.

### 🔐 AES-256-GCM Secret Store
All secrets — API keys, credentials, TLS private keys — stored in an encrypted secret store using AES-256-GCM authenticated encryption (`secrets.enc`). Immune to offline extraction without the master key.

### 🌐 TLS on All Endpoints
Full TLS encryption on all communication channels:

| Endpoint | Port |
|----------|------|
| API Server | `8443` |
| Dashboard | `8444` |
| Live Data Bridge | `8445` |

Certificate management via `server.crt / server.key` with CA-signed cert support.

---

## 🌐 Network Security & Zero Trust

### 🔐 ZTNA Policies
Zero Trust Network Access policies for all customer applications. Every access request is authenticated and authorized regardless of network location — replacing legacy VPN-based perimeter models.

### 🏢 DepartmentManager (Multi-Tenant)
Multi-tenant isolation engine for MSP and MSSP deployments. Each department or customer gets a fully isolated security environment within the same Sentinel installation — enabling managed service delivery at scale.

### ♻️ HAController (High Availability)
High-Availability controller for failover support. Ensures Sentinel remains operational during hardware failure, maintenance windows, or network disruptions — critical for government and production environments.

### 📊 Live Dashboard
Web-based management dashboard on **port 8444**. Shows real-time threat data, agent status, geo-block map, honeypot triggers, and health indicators. All settings configurable through the UI with no CLI required.

### 🔧 sentinel-ctl CLI
Full command-line control interface:

```bash
sudo sentinel-ctl status     # View system status
sudo sentinel-ctl logs       # Stream live logs
sudo sentinel-ctl restart    # Restart all services
sudo sentinel-ctl stop       # Graceful shutdown
sudo sentinel-ctl test       # Run 9-point health check
sudo sentinel-ctl dashboard  # Open web dashboard
```

Headless operation supported via `QT_QPA_PLATFORM=offscreen` for server environments.

### 🔌 API Bridge
Live data API bridge on **port 8445** feeds real-time telemetry to the dashboard and SIEM connectors. Runs as a dedicated systemd service with auto-restart on failure. Health endpoint available at `/api/v1/health`.

---

## 📡 SIEM Integration

Native connectors for **4 enterprise SIEM platforms**, configurable directly from the dashboard Settings tab — no config file editing required.

| Platform | Type |
|----------|------|
| **Splunk** | Enterprise SIEM |
| **Elastic** | ELK Stack / SIEM |
| **QRadar** | IBM Security |
| **Microsoft Sentinel** | Azure Cloud SIEM |

Log forwarding begins immediately after configuration. The `sentinel.log` main log feeds all SIEM connectors in real-time via the API bridge on port 8445.

---

## ✅ 9-Point Automated Health Check

Run `sudo sentinel-ctl test` at any time to execute all 9 checks. Full validation completes in seconds.

```
[OK] Sentinel service active
[OK] API responding on :8443
[OK] Dashboard responding on :8444
[OK] JA4+ database loaded: 308,683 fingerprints
[OK] nftables: 47 rules loaded
[OK] Suricata IDS process running
[OK] Kernel hardening active (ASLR=2, SYN=1)
[OK] Anthropic API key configured
[OK] Live data API bridge responding (:8445)
```

Any failing check outputs a diagnostic message pointing to the relevant troubleshooting section.

---

## ⬡ All 42 Engine Subsystems

<details>
<summary><strong>Click to expand full subsystem list</strong></summary>

**AI Layer**
- AI Commander (Opus 4.7)
- Vulnerability Agent (Sonnet 4.6)
- Anomaly Agent (Sonnet 4.6)
- Malware Agent (Sonnet 4.6)
- Triage Agent (Sonnet 4.6)
- Monitor Agent (Haiku 4.5)
- PenTesterAgent (Opus 4.7 · Every 15 min)
- RedTeamHunter (Opus 4.7 · Every 1 hour)

**Sovereign AI**
- Ollama 70B Local AI
- MLTrainer (Weekly)
- Teaching JSONL Logger

**Detection**
- JA4+ Fingerprint Engine
- Suricata IDS
- nftables Firewall
- eBPF / XDP Processor
- Geo-Block Engine
- Rate-Limiter
- Auto-Ban System

**Deception**
- SSH Honeypot
- HTTP Honeypot
- FTP Honeypot
- DeceptionEngine
- Honeytoken Deployer

**Hardening**
- AES-256-GCM SecretStore
- AppArmor Sandboxer
- auditd Audit Trail
- fail2ban
- Kernel Hardener (32 sysctls)
- TLS Certificate Manager

**Automation**
- CISA KEV Auto-Patcher
- ZTNA Policy Engine
- DepartmentManager (Multi-tenant)
- HAController (Failover)

**Management**
- Live Dashboard (Port 8444)
- API Server (Port 8443)
- Live Data Bridge (Port 8445)
- sentinel-ctl CLI

**SIEM Connectors**
- Splunk SIEM Connector
- Elastic SIEM Connector
- QRadar SIEM Connector
- MS Sentinel Connector

**Monitoring**
- 9-Point Health Check Engine

</details>

---

## 📊 Competitive Comparison

| Capability | SentinelFirewall v5 | Palo Alto | Fortinet | Check Point | Sophos |
|-----------|---------------------|-----------|----------|-------------|--------|
| Engine Subsystems | **42** | ~15 | ~12 | ~14 | ~8 |
| AI Commander (Multi-Agent) | ✅ Opus 4.7 + 5 agents | ◑ Partial ML | ◑ Partial AI | ✗ | ✗ |
| Sovereign / Offline AI (On-Prem LLM) | ✅ Ollama 70B | ✗ | ✗ | ✗ | ✗ |
| Automated Red-Teaming (15 min cycle) | ✅ | ✗ | ✗ | ✗ | ✗ |
| CISA KEV Auto-Patch (< 15 min) | ✅ | ✗ | ✗ | ✗ | ✗ |
| JA4+ Fingerprinting (308K sigs) | ✅ 308,683 | ◑ Partial | ✗ | ◑ Partial | ✗ |
| Built-in IDS (Suricata) | ✅ | ✗ | ◑ Partial | ✗ | ✗ |
| Deception / Honeypots | ✅ 3 honeypots + tokens | ✗ | ✗ | ✗ | ✗ |
| Source Code Ownership | ✅ Full IP transfer | ✗ | ✗ | ✗ | ✗ |
| SIEM Integration | ✅ 4 platforms | ✅ | ✅ | ✅ | ◑ |
| ZTNA Policies | ✅ | ✅ | ✅ | ✅ | ✗ |
| Multi-Tenant (MSP/MSSP) | ✅ DepartmentManager | ✅ | ✅ | ◑ | ✗ |
| Kernel Hardening (32 sysctls) | ✅ | ✗ | ✗ | ✗ | ✗ |
| 5-Minute Install / Single Command | ✅ | ✗ Days/weeks | ✗ Days | ✗ Days | ✗ |
| **Approx. Annual Cost** | **~$12,000/yr** | $300K–$2M | $200K–$800K | $250K–$1M | $50K–$200K |

---

## 🚀 Deployment & System Requirements

### Minimum Requirements
| Component | Minimum |
|-----------|---------|
| OS | Ubuntu 24.04 LTS |
| RAM | 8 GB |
| Disk | 50 GB |
| CPU | 4 Cores |
| Network | 1 Gbps NIC |
| Access | Root |

### Recommended (with Ollama 70B)
| Component | Recommended |
|-----------|-------------|
| OS | Ubuntu 24.04 LTS |
| RAM | 64 GB |
| Disk | 200 GB SSD |
| CPU | 8+ Cores |
| Network | 10 Gbps SFP+ |
| GPU | Optional (5–10× AI speed) |

### Installation

```bash
sudo bash install.sh
```

The installer auto-detects network interface, home CIDR, and OS variant (Ubuntu vs Kali). It asks only **6 questions** with sensible defaults. Full deployment including all 42 subsystems completes in:

- **5 minutes** — without Ollama
- **30+ minutes** — with the 70B model download

---

## 📜 License

Source code is provided to the customer under a **negotiated commercial license**. Full IP transfer available.

---

*42 SUBSYSTEMS · 6 AI AGENTS · 308,683 JA4+ FINGERPRINTS · 15-MIN RED-TEAM · <15-MIN KEV PATCH · ~$12,000/YR*
