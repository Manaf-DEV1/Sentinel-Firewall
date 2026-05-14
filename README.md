<div align="center">
Firewall is still under devoloping 

         
```
███████╗███████╗███╗   ██╗████████╗██╗███╗   ██╗███████╗██╗
██╔════╝██╔════╝████╗  ██║╚══██╔══╝██║████╗  ██║██╔════╝██║
███████╗█████╗  ██╔██╗ ██║   ██║   ██║██╔██╗ ██║█████╗  ██║
╚════██║██╔══╝  ██║╚██╗██║   ██║   ██║██║╚██╗██║██╔══╝  ██║
███████║███████╗██║ ╚████║   ██║   ██║██║ ╚████║███████╗███████╗
╚══════╝╚══════╝╚═╝  ╚═══╝   ╚═╝   ╚═╝╚═╝  ╚═══╝╚══════╝╚══════╝
```

### **FIREWALL**

*The world's first AI-native, self-pentesting, autonomous firewall*

---

![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Kali%20%7C%20Ubuntu-black?style=for-the-badge&logo=linux&logoColor=white)
![AI](https://img.shields.io/badge/AI-Claude%20Opus%204.7%20%2B%20Sonnet%204.6%20%2B%20Haiku%204.5-red?style=for-the-badge&logo=anthropic&logoColor=white)
![Engines](https://img.shields.io/badge/Engines-58%20Active-orange?style=for-the-badge)
![Agents](https://img.shields.io/badge/AI%20Agents-8-crimson?style=for-the-badge)
![Target](https://img.shields.io/badge/Target-Government%20%26%20Enterprise-darkred?style=for-the-badge)
![Author](https://img.shields.io/badge/Author-Manaf%20AL--Dulaimi-white?style=for-the-badge)
![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-black?style=for-the-badge)

</div>

---
SentinelFirewall 
========================

© 2026 Manaf AL-Dulaimi. All rights reserved.


What this is
============

SentinelFirewall is a Linux-native, headless security platform written
in modern C++ (C++17, Qt 6). It is positioned somewhere between a
traditional next-generation firewall, an XDR platform, a small SOC
automation suite, and a research tool for AI-driven defense.

It runs on Kali Rolling, Debian, and Ubuntu, and is built and tested
against government and industrial threat models including energy,
financial sector, and government enterprise environments.

What makes it different from a standard firewall is that the platform
combines several capabilities most products keep separate:

  - Packet filtering and rule enforcement (the firewall part)
  - Network detection and response (the IDS / IPS part)
  - Endpoint detection and response (the EDR part)
  - Extended detection and response with cross-engine correlation (the
    XDR part)
  - Live malware analysis with disassembly and emulation (the sandbox
    part)
  - User and entity behavior analytics (the UEBA part)
  - Forensic timeline reconstruction with STIX 2.1 and OpenIOC export
  - Multi-cloud security posture polling (AWS GuardDuty + Azure
    Defender + GCP SCC) with bidirectional alert forwarding
  - Zero Trust Network Access broker with JWT, posture checks, and
    continuous reauthentication
  - Sklearn machine learning training fed live by detection engines,
    with k-fold cross-validation and PSI drift detection
  - Eight cooperating AI agents (Claude Opus 4 commander plus seven
    role-specialized agents) that look for new attack methods and
    apply patches autonomously
  - SHA-256 audit chain over post-exploit findings for tamper detection
    (forensic-grade, useful for regulatory audit)
  - Active deception layer (DeceptionEngine) with 17 honeytoken types,
    automatic tripwire detection, and three-way alert forwarding

The code is currently ~35,000 lines of C++ spread across 62 engines
plus a Python dashboard, a Python ML helper, an eBPF XDP program, and
a Python Suricata bridge. It compiles to a single binary and lives at
/opt/sentinel/sentinel as a systemd-managed daemon.

This README is the master reference for what each engine does.


High-level architecture
========================

  +---------------------------------------------------------------+
  |                       Web Dashboard                            |
  |  (Python Flask app on :8444 — index.html + dashboard_api.py)  |
  +---------------------------------------------------------------+
                              ^
                              |  REST API on :8443
                              |
  +---------------------------------------------------------------+
  |                       APIServer (300 LOC)                     |
  +---------------------------------------------------------------+
                              ^
                              |
  +---------------------------------------------------------------+
  |              FirewallEngine (1610 LOC) — orchestrator          |
  |  - Loads, starts, stops 62 engines                             |
  |  - Wires shared services (XDR / Timeline / PostExploit / ML)  |
  |  - Exposes getters for every engine                            |
  +---------------------------------------------------------------+
              |                |                |
              v                v                v
        +----------+    +-----------+   +-----------+
        |   AI     |    | Detection |   |  Response |
        |  layer   |    |  layer    |   |   layer   |
        +----------+    +-----------+   +-----------+
        AIOrchestrator   IDS, EDR,        AutoIsolation,
        OllamaBridge     Beacon, XDR,     DeviceController,
        MLTrainer        Container,        CommanderAutoFix,
        PenTesterAgent   ZTNA, DLP, ...    SOAREngine

                              |
                              v
              +----------------------------------+
              | Forensic + Audit + Cloud layer   |
              | ForensicTimeline, PostExploit,    |
              | CloudConnector, SIEMConnector     |
              +----------------------------------+


The eight AI agents
====================

The brain of the platform is eight cooperating LLM agents organized
into a council, all driven by AIOrchestrator (1276 LOC, STRONG). They
run on different cadences and feed each other.

1. Commander (Claude Opus 4)
   The decision authority. Receives summarized state every 15 minutes,
   reviews findings from the other seven agents, and decides what to
   actually patch, block, or escalate. It can refuse to act if the
   evidence is weak. Talks to the API server through OllamaBridge or a
   direct Anthropic API call.

2. Vulnerability Specialist (LLM student trained from Commander)
   Reads VulnerabilityFeedEngine data (NVD, Vulners, MITRE CISA KEV,
   CVSS), correlates with installed software via D3FENDMapper, and
   suggests prioritized mitigations.

3. Anomaly Specialist
   Reads UEBAEngine baselines and BeaconDetector verdicts. Identifies
   behavior that looks weird even if no signature matches. Often the
   first to flag insider-threat scenarios.

4. Malware Specialist
   Reads PolymorphicAnalyzer + DownloadInspector + IDAEngine outputs.
   Decides if a sample is malicious based on disassembly, packing,
   YARA, and Capstone-disassembled control flow.

5. Triage Specialist
   Reads XDREngine correlated incidents and produces a one-paragraph
   summary an analyst can act on without diving into raw events.

6. Red Team Specialist
   Driven by RedTeamHunter. Continuously tries to find attack paths
   the defender hasn't covered. Reports gaps as PostExploitFindings.

7. PenTester Specialist
   Driven by PenTesterAgent. Runs live offensive tests against the
   firewall's own perimeter (port scans, beacon emulation, fileless
   payload tests) to confirm detection rules work. Findings feed back
   into RuleEngine.

8. Monitor Specialist
   Watches HealthMonitor + AlertManager. If an engine fails or runs
   hot (rate anomaly), Monitor proposes a remediation.

A ninth specialist called commander-autofix is actually a trained
local Ollama student model (registered in MLTrainer), which lets the
firewall keep operating in autonomous mode even when the Anthropic API
isn't reachable.

The eight agents talk through the KnowledgeBase (428 LOC, GOOD), which
acts as shared scratch memory. Their decisions get sequenced and
applied by CommanderAutoFix (418 LOC, GOOD).


Engine catalog — all 62 engines
================================

Below, every single engine is documented. The number in parentheses
is the line count (header + .cpp combined) and the tier.


Orchestration and core
-----------------------

FirewallEngine (1610, STRONG)
  The root object. Owns one instance of every other engine. Holds
  member pointers like m_ueba, m_xdr, m_timeline, m_postExploit, etc.
  Exposes public getters so engines can find each other. Provides
  initSubsystems() which constructs, wires dependencies, and starts
  every engine in the right order. Also owns the lifecycle:
  shutdown(), reload(), and config reload.

AIOrchestrator (1276, STRONG)
  Implements the eight-agent council described above. Manages prompt
  templates, agent cadence timers, conversation summarization, and
  the Commander's authority over patch application. Talks to OllamaBridge
  for local LLMs and the direct Anthropic API for Commander Opus 4.

HealthMonitor (384, GOOD)
  Polls every engine every 30 seconds. Calls isRunning() + stats() and
  flags any engine that's stuck, leaking, or emitting suspiciously
  many events per minute (rate anomaly). Exports health metrics to
  the dashboard and emits warnings to AlertManager.

KnowledgeBase (428, GOOD)
  In-memory shared state, mutex-protected. The AI agents and the
  detection engines write summaries here and read from each other.
  Has TTL on entries to prevent unbounded growth.

SecretStore (300, OK)
  Encrypted at-rest secret storage using a master key in
  /etc/sentinel/master.key (mode 0600). Holds API keys for Anthropic,
  Ollama, cloud credentials, JWT signing keys, etc. Exposed as
  get(key) / set(key, value). All other engines fetch their secrets
  from here instead of hardcoding.

HAController (421, GOOD)
  High availability primary/standby. Pings peers, exports state
  snapshots, and on primary failure promotes a standby with the most
  recent state. Heartbeat over UDP unicast.

APIServer (413, GOOD)
  HTTP/HTTPS server on port 8443. Serves /api/* endpoints used by the
  Python dashboard and any third-party SIEM integration. JSON in,
  JSON out. JWT-protected for non-localhost.

AlertManager (390, GOOD)
  Centralizes severity-tagged alerts from every engine. Deduplicates
  within a window, routes to syslog / email / webhook based on
  configured severity floor, and pushes to SIEMConnector.

SignatureUpdater (330, GOOD)
  Polls upstream sources (YARA, Suricata-rules, MISP, Sigma) on a
  schedule, validates signatures, and pushes updates atomically.

SOAREngine (413, GOOD)
  Stitches together playbooks (YAML). A trigger fires, the playbook
  runs a sequence of actions across engines (isolate device, query
  threat intel, post Slack alert, page on-call). 14 built-in
  playbooks covering common incident classes.

main (93, core entry)
  Boot logic. Sets up Qt logger, parses CLI flags, instantiates
  FirewallEngine, and enters the Qt event loop. Handles SIGTERM /
  SIGINT for clean shutdown.


Network detection and inspection
--------------------------------

IDSEngine (788, STRONG)
  Suricata-rule-format IDS. Reads .rules files, parses them into
  internal rule trees, evaluates against live packet streams (via
  AF_PACKET in headless mode, optionally via DPDK or eBPF in
  high-throughput mode). Supports content matching, regex, byte
  jumps, and protocol decoders for TCP, UDP, ICMP, HTTP, DNS, TLS,
  SMB, FTP.

BeaconDetector (944, STRONG)
  Detects command-and-control beaconing — periodic phone-home traffic
  that malware uses to receive instructions. Real implementation:
  per-flow time-series analysis with FFT autocorrelation peak
  detection, jitter ratio, chi-squared distribution test for
  randomness, DGA scoring (entropy + n-gram + bigram-frequency), and
  DNS tunneling indicators. Outputs C2Beacon records with verdict:
  BENIGN, MONITOR, SUSPICIOUS, HIGH_CONFIDENCE_C2.

DNSFirewall (285, OK)
  Sinkhole-style DNS filter. Maintains a blocklist of known-malicious
  domains (from VulnerabilityFeedEngine + commercial threat intel),
  resolves queries locally for those, and forwards others to upstream.
  Logs every blocked query to ForensicTimeline.

GeoBlocker (291, OK)
  IP-range-to-country lookup using MaxMind GeoLite2 database. Blocks
  inbound or outbound traffic from configurable country sets.

VPNTorDetector (495, GOOD)
  Identifies traffic going through known VPN providers and Tor exit
  nodes. Combines IP reputation lists with TLS fingerprint analysis.
  Useful for ZTNA posture and for spotting privilege escalation that
  uses anonymizers.

EncryptedTrafficAnalyst (499, GOOD)
  Analyzes TLS metadata (no decryption needed): JA4 fingerprints,
  certificate chain validity, SNI patterns, session duration,
  bytes-in / bytes-out ratio. Detects C2-over-TLS even when content
  is encrypted. Pairs with a Python ML model trained on labeled
  TLS flows via train_eta_model.py.

SSLInspector (592, STRONG)
  Optional TLS interception (man-in-the-middle, requires CA cert
  trusted by clients) for deep inspection of HTTPS payloads. Off by
  default. When on, decrypted payloads flow into DLP, WAF, and YARA.

WAFEngine (290, OK)
  HTTP request firewall. OWASP CRS-compatible rule format with custom
  extensions. Blocks SQL injection, XSS, path traversal, command
  injection, deserialization attacks. Has machine-learning-based
  anomaly mode that flags requests deviating from learned baselines.

JA4Fingerprinter (537, STRONG)
  Generates JA4 / JA4T / JA4S / JA4H fingerprints from TLS handshakes
  and HTTP requests. Lookup database includes Chrome, Firefox, Safari,
  curl, python-requests, Tor, Cobalt Strike default profile, Sliver,
  Mythic Apollo, Empire, Metasploit Meterpreter. C2 framework
  fingerprints fire anomaly alerts immediately.


Endpoint detection and control
------------------------------

EDREngine (389, GOOD)
  Endpoint agent coordinator. Receives /proc snapshots, file system
  change notifications, and syscall traces from agents installed on
  protected hosts. Correlates with network signals from IDS / Beacon.

DeviceScanner (939, STRONG)
  Network-wide device discovery. Uses ARP, mDNS (Bonjour), SSDP, SNMP,
  NetBIOS, and active probing (Nmap-style SYN scans on common ports).
  Builds the device inventory that DeviceFingerprint then enriches.

DeviceFingerprint (1573, STRONG)
  Multi-signal device classification — easily the most thorough engine
  in the platform. Combines eight identification channels:
    - MAC OUI vendor lookup (100+ vendor prefixes)
    - p0f-style TCP signature (TTL + window size + MSS + options)
    - JA4 / JA3 TLS fingerprint (16+ known prefixes including C2
      frameworks)
    - HTTP User-Agent regex parsing
    - mDNS / Bonjour service mapping (25+ service types)
    - SNMP sysDescr / sysObjectID parsing
    - Open-port classification (14 ports: 9100 printer, 502 Modbus
      PLC, 102 Siemens S7, 44818 Rockwell EtherNet/IP, 47808 BACnet,
      1883 MQTT, etc.)
    - Hostname pattern matching ("plc", "scada", "hmi", "camera")
  Per-device risk score (0-100) with anomaly detection: vendor-OS
  mismatch (Apple MAC + Windows OS = MAC spoof), printer with admin
  ports, end-of-life Windows, OT device exposing HTTP. 21-class device
  enum (workstation / server / laptop / mobile / printer / camera /
  NAS / router / firewall / IoT / smart-home-hub / PLC-SCADA / HMI /
  historian / VM / container / medical / POS / SmartTV / console /
  unknown) and 12-OS-family enum.

DeviceController (1017, STRONG)
  Active device control. Six isolation modes:
    BLOCK_ALL          — full input + forward drop
    BLOCK_INTERNET_ONLY — forward drop to non-LAN destinations
    BLOCK_LAN_ONLY     — drop forward everywhere
    BANDWIDTH_LIMIT    — tc/qdisc throttling at configurable kbps
    REDIRECT_TO_HONEYPOT — forward only to a honeypot IP
    VLAN_MOVE          — SNMP-based VLAN migration (stub for
                          managed-switch integration)
  Real remote actions via SSH (restart / shutdown / kill-pid),
  Wake-on-LAN with wakeonlan + etherwake fallback. Auto-expiry timer
  for time-limited quarantines. Records persisted to disk; survive
  daemon restart. nft handle tracking for clean rule removal.

WindowsDeviceAgent (821, STRONG)
  Remote management of Windows endpoints via WMI and PowerShell
  Remoting. Pull running processes, services, logged-on users,
  recently installed software, scheduled tasks, registry hives.
  Can also push actions: restart service, end task, restart machine.

NACController (416, GOOD)
  Network Access Control. Decides who can join the network and at
  what posture. Pairs with ZTNABroker for continuous verification.
  Integrates with 802.1X via RADIUS shim.

MemoryForensics (356, GOOD)
  Live memory analysis on protected hosts. Pulls process memory via
  /proc/PID/mem (Linux) or via WMI-injected agent (Windows). Looks
  for unbacked executable memory regions, suspicious DLL injection,
  unhooked syscall stubs, signs of process hollowing.


Malware analysis
----------------

DownloadInspector (888, STRONG)
  The malware analysis entry point. Watches HTTP / HTTPS / SMB / FTP
  downloads, computes SHA-256, checks YARA rules, sends to
  SandboxDetonator for behavioral analysis, and on verdict-malicious
  quarantines the file + adds nft block on the source IP. Captures
  IOCs (URLs, command-and-control IPs, dropped file names) and feeds
  them to ForensicTimeline + PostExploitLogger.

IDAEngine (889, STRONG)
  Disassembler and emulator. Uses Capstone for disassembly (x86_64,
  ARM, ARM64, MIPS) and Unicorn Engine for emulation in a controlled
  CPU sandbox. Reports control-flow graph, suspicious API call
  sequences (LoadLibrary then GetProcAddress = dynamic resolution
  pattern, CreateRemoteThread = injection, VirtualAllocEx + WriteProcessMemory
  pattern, etc.), packer detection (entropy analysis), and exit
  reasons (segfault, infinite loop, syscall, attempted shellcode
  execution).

PolymorphicAnalyzer (616, STRONG)
  Classifies samples as polymorphic, metamorphic, packed, fileless,
  LOLBin abuse, BYOVD (Bring Your Own Vulnerable Driver), AI-generated
  malware, or any combination. Computes maliciousScore and confidence,
  attaches MITRE technique IDs, generates YARA rules from observed
  behavior, and produces a recommendedAction. Output feeds into
  AICodeContextAnalyzer for explanation.

SandboxDetonator (526, STRONG)
  Behavioral sandbox runner. Detonates suspicious binaries inside a
  Firejail or systemd-nspawn container with no network, snapshots
  before/after filesystem state, captures dropped files, registered
  scheduled tasks (Linux: systemd timer / cron; Windows agent: task
  scheduler), network attempts (caught at the firewall), and child
  process tree.

AntiVirus (413, GOOD)
  Traditional signature-based scanning using ClamAV libraries and the
  YARA engine. Acts as a fast-path filter before the heavy analysis
  in PolymorphicAnalyzer + IDAEngine. Definitions auto-updated by
  SignatureUpdater.

YARARuleManager (299, OK)
  Owns the YARA rule database. Compiles rules to a single bytecode
  blob for performance. Hot-reloads on SignatureUpdater triggers.
  Exposes scan(buffer) and scan(file_path) for other engines.


AI and machine learning
-----------------------

MLTrainer (1060, STRONG)
  Sklearn training pipeline coordinator. Writes a Python helper
  (/var/lib/sentinel/sentinel_ml_helper.py) at startup and invokes it
  via QProcess with JSON stdin/stdout. Four helper actions:
    - train_isoforest: IsolationForest for anomaly detection
    - train_rf: RandomForest with k-fold cross-validation
    - train_dbscan: DBSCAN clustering for malware family ID
    - compute_psi: Population Stability Index for drift detection
  Real k-fold (k=5 default) yields real precision/recall/F1/accuracy
  numbers — not synthetic. Drift triggers automatic retrain. Three
  detection engines feed live training examples via
  addTrainingExample():
    - UEBAEngine    → ueba_anomaly (IsolationForest)
    - BeaconDetector → beacon_classifier (RandomForest)
    - PolymorphicAnalyzer → malware_rf (RandomForest)
  Plus nine LLM student models (Ollama) for the AI agent roles.

OllamaBridge (347, GOOD)
  HTTP wrapper for a local Ollama daemon serving open-weight LLMs
  (llama3.1:70b by default). Used by the AI agents as a fallback
  when the Anthropic API is unreachable. Also used to fine-tune the
  per-role student models created by MLTrainer.

PythonBridge (502, STRONG)
  Generic Python subprocess shim for any task that's easier in
  Python (sklearn ML, Suricata bridge, Pandas analytics, MISP API).
  Inputs JSON over stdin, reads JSON from stdout, handles timeouts
  and error propagation cleanly.

AICodeContextAnalyzer (573, STRONG)
  Reads a disassembled binary's control flow graph from IDAEngine and
  generates a human-readable explanation of what the code does using
  the Malware Specialist agent. Output looks like "this function
  resolves LoadLibrary then loads kernel32.dll then calls VirtualAllocEx
  with PAGE_EXECUTE_READWRITE — classic shellcode loader."

AIPentestDefense (765, STRONG)
  Detects when an attacker is using AI tools (e.g. LLM-generated
  exploit code, AI-assisted automation, AutoGPT pentest agents).
  Looks for telltale patterns: rapid sequence of distinct CVE
  exploitation attempts, code-like patterns in HTTP request bodies,
  unusually well-formatted payloads with comments. Pairs with
  RuleEngine to dynamically up-weight blocks against AI-driven
  attackers.


Adversary emulation (red team)
-------------------------------

PenTesterAgent (1128, STRONG)
  An offensive agent that continuously tests the firewall's own
  defenses. Performs port scans, attempts privilege escalation
  techniques against honeypot endpoints, runs Atomic Red Team
  scenarios (T1059 Command and Scripting Interpreter, T1086 PowerShell,
  T1078 Valid Accounts, etc.), and confirms each is caught. Findings
  feed into RuleEngine for gap remediation.

RedTeamHunter (888, STRONG)
  Looks for what the defender hasn't yet thought of. Combines current
  CVE feed data with deployment-specific configuration (asset list,
  device fingerprints, exposed ports) to identify under-defended
  attack paths. Posts each as a PostExploitFinding so CommanderAutoFix
  can prioritize patching them.


Threat intel and correlation
-----------------------------

XDREngine (904, STRONG)
  The cross-engine correlation graph. Receives XDREvent objects from
  seven detection engines via ingestStructuredEvent(). Each event has
  source, kill-chain-phase (per MITRE), MITRE tactics + techniques,
  IOCs (IP / hash / user / process / domain / JA4), and timestamps.
  Builds an entity-graph in real time: events sharing entities become
  connected nodes. Periodically runs connected-component analysis to
  emit Incident objects (a kill-chain over many events sharing
  evidence). Output streams to ForensicTimeline + AlertManager.

ForensicTimeline (1105, STRONG)
  The historical record. Every engine that records security-relevant
  events sends them here via recordEvent(). Maintains up to 50,000
  events (configurable) plus persistent JSONL on disk that survives
  restart. Capabilities:
    - Plaso CSV import (industry-standard DFIR timeline format)
    - STIX 2.1 export (proper bundle with Identity, Indicator, Report)
    - OpenIOC 1.1 (Mandiant) XML export
    - CEF export for ArcSight / QRadar / Splunk
    - Graph-based multi-actor incident reconstruction (BFS over
      entity-graph, similar to XDR but for historical data)
    - Time-skew detection (per-event ingest-vs-source skew, flags
      engines with clock drift)
    - Rate anomaly detection (engine emits > 100 events / minute)

SigmaHunter (340, GOOD)
  Sigma-format rule scanner. Sigma is a portable detection-rule
  language — rules written once translate to Splunk / Elastic /
  QRadar queries. SigmaHunter runs Sigma rules locally against
  ForensicTimeline events.

VulnerabilityFeedEngine (620, STRONG)
  Pulls NVD, MITRE CISA KEV, Vulners, and OSV feeds on a schedule.
  Stores CVE records keyed by CPE for lookup by D3FENDMapper. Tracks
  CVSS, CWE, EPSS exploitability score, and known-exploited-in-wild
  flag. New CRITICAL CVEs trigger an AI-agent review pass.

D3FENDMapper (182, THIN)
  Maps MITRE D3FEND defensive techniques onto the deployed Sentinel
  rule set. Useful for compliance reports: "which D3FEND techniques
  does this deployment cover?" Currently has the mapping logic but
  needs a hardening pass to add real coverage queries.

DarkwebMonitor (228, OK)
  Polls a configurable set of dark-web data sources (paste sites, Tor
  market forums, breach databases) for mentions of the protected
  organization's domain / IP ranges / employee emails. Real
  integration is via SecretStore-stored API keys for third-party
  services (HaveIBeenPwned, Recorded Future, Flashpoint).


Containers and cloud
--------------------

ContainerSecurity (1122, STRONG)
  Docker and Kubernetes runtime monitoring. Capabilities:
    - Polls dockerd / containerd / CRI-O for running containers
    - Trivy CVE scanning of container images on pull
    - Runtime drift detection (Falco-style: container started with
      capability X, now suddenly running pid as root)
    - Privileged-container flagging
    - Auto-pause container on critical Falco event
    - Risk score per container based on image age, CVE count,
      capabilities, mount points, exposed ports
    - Integration with k8s API for namespace/pod context

CloudConnector (1277, STRONG)
  Multi-cloud security posture ingestion. Real implementations for:
    - AWS GuardDuty + AWS SecurityHub (full SigV4 signing in pure
      Qt — no AWS SDK dependency; 4-step HMAC-SHA256 key derivation)
    - Azure Defender for Cloud + Azure Sentinel (OAuth2
      client_credentials with token caching)
    - GCP Security Command Center (real RS256 JWT signing via
      OpenSSL EVP_DigestSign)
  Bidirectional: not only ingests cloud findings, but pushes
  Sentinel-generated alerts UP to CloudWatch Logs, Azure Log
  Analytics workspace, and GCP Cloud Logging. Severity normalization
  across clouds. Persistence + de-duplication of findings.


Zero trust and deception
------------------------

ZTNABroker (1224, STRONG)
  Zero Trust Network Access broker. Issues short-lived JWTs to
  authenticated users, enforces posture checks (device fingerprint
  + EDR health + geo-location), and does continuous reauthentication
  (token re-verification every N minutes plus on every sensitive
  action). Risk-score per session aggregated from many signals.
  MFA challenge insertion on risk score escalation. Posture lookups
  flow back into DeviceFingerprint.

HoneypotEngine (1042, STRONG)
  Eleven-protocol honeypot rig. Listens on configurable ports for
  SSH, Telnet, FTP, HTTP, HTTPS, SMB, RDP, MSSQL, MySQL, Postgres,
  and Modbus. Captures credentials, fingerprints clients, and emits
  a HoneypotInteraction event for every attempted login. Any
  interaction is high-fidelity attacker-evidence: real users don't
  type credentials into a honeypot. Auto-blocks the source IP on
  the first credential attempt.

DeceptionEngine (1460, STRONG)
  Active deception layer. Plants realistic honeytoken artifacts across
  the filesystem, DNS, and HTTP canary infrastructure, then watches
  for any attacker interaction — read, write, or network request —
  and fires three-way alerts the moment a token is touched.

  17 honeytoken types:

    CREDENTIAL      Postgres-style DB connection string (user / pass /
                    host / port / db)
    API_KEY         Service-correct prefix: sk_live_ Stripe, sk- OpenAI,
                    SK Twilio, SG. SendGrid
    SSH_KEY         Full OpenSSH PEM with 24 lines of random base64 body
    AWS_KEY         AKIA... access key + 40-char base64 secret +
                    multi-profile credentials file
    GIT_TOKEN       ghp_ GitHub PAT in .netrc format
    SLACK_WEBHOOK   Full hooks.slack.com/services/T.../B.../... URL
    JWT             Real HS256 JWT with admin claims + 1-year expiry
    K8S_SA_TOKEN    192-char Kubernetes service account token
    DOCKER_TOKEN    dckr_pat_ in Docker config.json with base64 auth
    JIRA_TOKEN      ATATT... Atlassian API token in ini config
    DATABASE_URL    postgres / mysql / mongodb / redis connection strings
    DNS_CANARY      Unique subdomain.canary.sentinel.local FQDN
    WEB_BUG         Tracking URL canary.sentinel.local/trap/...
    HONEYFILE       Arbitrary decoy content with SHA-256 fingerprint
    BREADCRUMB      Fake admin console URL + default creds bait
    FAKE_ADMIN_SHARE Directory with admin_only.txt + backup_keys.txt
    FAKE_DOCUMENT   Markdown with embedded web bug tracking pixel

  Tokens are realistic. For example, the AWS credentials file is
  multi-profile with real region values and AKIA-prefixed keys.
  Attackers who grep ^AKIA will find it.

  Tripwire detection (no external code required):

    QFileSystemWatcher    Any write / modify to a decoy path. Fires
                          immediately.
    Atime polling         Reads — stat() each decoy every 30 s, compare
    (every 30 s)          atime to last-known. Atime advance = unauthorized
                          read. Requires filesystem NOT mounted noatime.
    DNS canary            DNSFirewall hook → reportDNSQuery() on any query
                          for *.canary.sentinel.local
    Web bug               SSLInspector / WAF hook → reportWebHit() on any
                          HTTP request to the tracking URL

  Token rotation: rotateExpiredTokens() replaces untriggered tokens
  older than 7 days (configurable). Catches attackers who exfiltrated
  a credential but haven't used it yet — by the time they do, it has
  been rotated.

  On every trigger, three-way alert forwarding fires simultaneously:
    - PostExploitLogger → CRITICAL finding with MITRE T1552
      (Unsecured Credentials) + T1083 (File Discovery), attack chain
      explanation, and recommended fix
    - XDREngine → XDREvent with phase=CREDENTIAL_ACCESS,
      tactics=[CredentialAccess, Discovery]
    - ForensicTimeline → audit-grade event record with full metadata

  Default plant on first start: /var/lib/sentinel/decoys/ is
  automatically populated with 16 file-based decoys + 2 DNS canaries
  + 1 web bug. Survives restart via JSONL persistence.


OT and industrial
-----------------

OTProtocolParser (475, GOOD)
  Parses industrial protocols common in OT deployments: Modbus,
  Siemens S7Comm, EtherNet/IP (Rockwell), BACnet, DNP3,
  IEC 60870-5-104. Flags dangerous PLC commands (write-to-coil,
  force-coil, stop-PLC, download-program) for whitelist-only
  enforcement on OT segments.


Data loss prevention
--------------------

DLPEngine (436, GOOD)
  Content inspection for sensitive data leaving the network: credit
  cards (PAN, with Luhn check), SSNs, national IDs, IBANs, passport
  numbers, classified document markers. Operates on decrypted HTTP
  via SSLInspector or on plaintext protocols. Per-rule action:
  alert / quarantine / block.

EmailDLP (274, OK)
  Email-specific extension of DLP. Hooks into SMTP-out via Postfix
  milter interface. Same content rules as DLPEngine plus
  attachment-extension blacklists and password-encrypted-archive
  detection.


Response and remediation
------------------------

AutoIsolation (378, GOOD)
  High-level isolation orchestration. When XDR or PostExploitLogger
  emits a critical incident, AutoIsolation decides what to isolate
  (one host? VLAN? subnet?) and delegates to DeviceController for
  the actual nft + tc + SSH actions. Tracks isolation history.

CommanderAutoFix (418, GOOD)
  The autonomous patcher. Reads PostExploitLogger.unfixedFindings(),
  presents prioritized list to Commander Claude, applies approved
  patches as nft rule additions, sysctl changes, config edits, or
  systemctl service restarts. Logs every action with diff so
  rollback is possible.

PostExploitLogger (1024, STRONG)
  The vulnerability ledger. Every finding from every engine that
  represents a real or suspected weakness goes here. Capabilities:
    - SHA-256 audit chain across findings (tamper detection: any
      manual edit to the JSON breaks the chain and is reported at
      the exact broken index)
    - STIX 2.1 export with Identity + Report + Indicator objects
    - OpenIOC 1.1 (Mandiant) XML export
    - CSV export for compliance audits
    - De-duplication by ID
    - Bulk fix operations (markFixedByPhase, markFixedBySeverity)
    - Atomic persistence (.tmp + rename for crash safety)
    - XDR forwarding for HIGH / CRITICAL findings


Behavioral analysis and rules
------------------------------

UEBAEngine (462, GOOD)
  User and entity behavior analytics. Builds per-entity behavioral
  baselines (login times, source IPs, accessed resources, command
  patterns) and computes anomaly scores when current behavior
  deviates. Feeds the ML training pipeline.

RuleEngine (533, STRONG)
  Generic policy enforcement layer. Reads policy YAML from
  /etc/sentinel/rules/, compiles to predicates, evaluates against
  inbound events. Allows the firewall to express "if engine X reports
  Y, then engine Z does W" without recompiling C++.

CheckpointEngine (663, STRONG)
  State snapshotting. Periodically captures the configuration and
  active-rule state of every engine for disaster recovery and
  audit-trail compliance. Snapshots are signed with the master key
  and can be replayed for forensic analysis.


SIEM and integrations
---------------------

SIEMConnector (312, OK)
  Generic outbound bridge to enterprise SIEMs: Splunk HEC, Elastic
  Common Schema, Microsoft Sentinel, IBM QRadar, ArcSight CEF.
  Configurable format and destination. Backed by AlertManager.


eBPF and high performance
-------------------------

EBPFLoader (377, GOOD)
  Loads the XDP (eXpress Data Path) program from ebpf/xdp_firewall.c
  into the kernel for line-rate packet filtering before the packet
  reaches user space. Handles map updates (allow / deny lists,
  rate limits) from RuleEngine.


Admin
-----

DepartmentManager (208, THIN)
  Multi-tenancy primitive. Tags users, devices, and rules with a
  department (e.g. Finance / OT / GuestWiFi). Rule visibility and
  alert routing scoped by department. Needs hardening pass to add
  cross-department audit policies.


Supporting components (not Qt engines)
=======================================

eBPF program (ebpf/xdp_firewall.c)
  Kernel-side XDP filter. Compiled to BPF bytecode and pinned to a
  network interface. Provides line-rate packet filtering and per-CPU
  hash maps used by IDSEngine and RuleEngine for the hot path.

Python ML helper (/var/lib/sentinel/sentinel_ml_helper.py)
  Created at runtime by MLTrainer. Implements sklearn calls
  (IsolationForest, RandomForestClassifier, DBSCAN) and PSI drift
  calculation. Invoked via QProcess.

Suricata integration (python/suricata_integration.py)
  Bridge that feeds Suricata eve.json events into IDSEngine over a
  named pipe or local TCP.

Train-ETA model (python/train_eta_model.py)
  Offline training pipeline for the EncryptedTrafficAnalyst model.
  Reads labeled flow data, trains a small classifier, exports to
  ONNX which the C++ side then loads.

Sentinel main wrapper (python/sentinel_main.py)
  Legacy compatibility shim. Some older deployments call this. New
  deployments don't need it.

Web dashboard (dashboard/index.html + dashboard_api.py)
  Flask app on port 8444. Talks to the C++ REST API on 8443.
  Provides real-time view of engine status, recent events, active
  incidents, device inventory, threat intel feed status, AI agent
  conversation history.


Build and install
==================

Required dependencies on Kali Rolling (also works on Debian 12 +
Ubuntu 24.04 LTS):

  apt install build-essential cmake qt6-base-dev qt6-base-private-dev
              libqt6sql6-sqlite libcap-dev libnetfilter-conntrack-dev
              libnetfilter-queue-dev libpcap-dev libnfnetlink-dev
              libssl-dev libcapstone-dev libunicorn-dev libbpf-dev
              clang llvm bpftool python3 python3-pip yara libyara-dev
              clamav clamav-daemon nftables tc wakeonlan etherwake
              suricata zeek

  pip3 install --break-system-packages scikit-learn numpy pandas
              flask requests boto3 azure-identity azure-mgmt-security
              google-cloud-securitycenter

Then:

  cd ~/Desktop/SentinelV3
  sudo bash install.sh

install.sh builds the C++ project, copies the binary to
/opt/sentinel/sentinel, installs the systemd service unit, drops
config templates into /etc/sentinel/, and creates working directories
under /var/lib/sentinel/ and /var/log/sentinel/.

To start:

  sudo systemctl enable --now sentinel
  sudo journalctl -fu sentinel

To see logs:

  sudo tail -f /var/log/sentinel/sentinel.log


Configuration
==============

Primary config: /etc/sentinel/sentinel.json

This is the main config file. It controls which engines start, their
poll intervals, and which interfaces to bind. See INSTALL_GUIDE.md
for the field-by-field reference.

Secrets: /etc/sentinel/secrets.json (loaded by SecretStore, mode 0600)

Holds API keys for Anthropic, cloud credentials, JWT signing keys,
upstream threat intel keys. Encrypted at rest with the master key in
/etc/sentinel/master.key.

Rules: /etc/sentinel/rules/*.yaml

RuleEngine's policy YAML. Live-reloaded.

YARA rules: /etc/sentinel/yara/

Loaded by YARARuleManager.

Suricata rules: /etc/sentinel/suricata-rules/

Loaded by IDSEngine.

Master key: /etc/sentinel/master.key (32-byte random, mode 0600)

Generated by install.sh on first run. Used by SecretStore for AES-256
encryption of all secrets at rest. Don't lose this — you'd have to
re-enter every secret.


Operations
===========

Daily

  - dashboard at http://your-ip:8444 — overview, incidents, devices
  - sudo journalctl -fu sentinel — live log
  - /var/log/sentinel/sentinel.log — persistent log (logrotate handles
    rotation)

Common tasks

  Block a country:
    Edit /etc/sentinel/geoblock.json, restart GeoBlocker via dashboard

  Add a YARA rule:
    Drop file into /etc/sentinel/yara/, SignatureUpdater hot-reloads

  Quarantine a device:
    Dashboard → Devices → select → Actions → Quarantine
    Or directly: nft + DeviceController

  Export forensic timeline to STIX:
    Dashboard → Timeline → Export STIX 2.1
    Output bundle goes to /var/lib/sentinel/exports/

  Verify audit chain (tamper check):
    Dashboard → Findings → Verify Audit Chain
    Or restart sentinel — verification runs on every start.


Roadmap and honest assessment
==============================

What's done well

  62 engines, ~35,000 lines, build stable across 14 hardening
  sessions in a row. 25 STRONG engines (real implementations).
  Active deception layer with 17 honeytoken types and automatic
  tripwire detection. Multi-protocol honeypot, real disassembly +
  emulation pipeline, real ML training with proper k-fold CV, real
  cloud security integrations including AWS SigV4 from scratch and
  GCP RS256 JWT signing via OpenSSL. Real graph-based XDR. Forensic
  timeline with STIX 2.1 / OpenIOC / Plaso. Tamper-evident audit
  chain on vulnerabilities.

What's still placeholder

  Two THIN engines remain:
    - DepartmentManager (multi-tenancy)
    - DarkwebMonitor (commercial-feed integration)

  D3FENDMapper is at OK tier — wired and functional but needs deeper
  coverage queries.

  These have the right interfaces and are wired into FirewallEngine,
  but they need a hardening pass to gain real depth.

What this is not yet

  This is a strong technical foundation. It is not yet a
  production-deployed product. Specifically:

    - Has not been through a third-party security audit
    - Has not been deployed in a real customer environment for an
      extended period
    - Has no commercial support contract or SLA
    - Has no formal compliance certification (ISO 27001, SOC 2, CC,
      common-criteria-eval)

  Going from "works on Kali" to "deployed in production" requires
  bug fixes from real traffic, hardening against real adversaries,
  legal contracts, support staffing, and customer trust built over
  months.

What's coming next

  v9.14: DepartmentManager hardening
  v9.15: DarkwebMonitor hardening
  v9.16: D3FENDMapper hardening

  After v9.16, all 62 engines will be at GOOD or STRONG tier (no
  THIN, no STUB). The line count will be roughly 36,000.

  After that, the focus shifts from feature-adding to:
    - Real deployment in a test environment
    - Performance profiling under realistic packet loads
    - Documentation for operators
    - Compliance work


License and attribution
=======================

© 2026 Manaf AL-Dulaimi. All rights reserved.

This codebase is the personal work product of Manaf AL-Dulaimi. It
incorporates the following open-source dependencies under their
respective licenses: Qt 6 (LGPL v3), Capstone (BSD-3), Unicorn Engine
(GPLv2), LIEF (Apache 2.0), libbpf (LGPL v2.1 / BSD-2), OpenSSL
(Apache 2.0), libpcap (BSD-3), nftables tools (GPL), tc / iproute2
(GPL), Suricata (GPLv2), YARA (BSD-3), ClamAV (GPLv2).

Trademarks referenced (Cisco, Apple, Samsung, AWS, Azure, GCP, Mandiant,
Kali, Anthropic, Claude, Ollama, etc.) are the property of their
respective owners and are referenced only for technical interoperability
purposes.

</div>
