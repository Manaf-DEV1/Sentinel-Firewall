# SentinelFirewall

![C++](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white)
![Qt](https://img.shields.io/badge/Qt-6-41CD52?logo=qt&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Linux%20(Kali)-1793D1?logo=linux&logoColor=white)
![Status](https://img.shields.io/badge/status-prototype-orange)
![Build](https://img.shields.io/badge/build-CMake%20%2B%20Ninja-064F8C?logo=cmake&logoColor=white)

> 🎓 **A student project, built in the open.** I'm an undergraduate
> student teaching myself systems programming and security engineering by
> building something real and hard. This repo is me learning in public: the honest test results, the things that work, and — deliberately —
> the things that don't yet. I'd rather show an accurate picture of a real
> prototype than a polished pitch for something it isn't. Advices from people
> who know this field better than I do is genuinely welcome.

A multi-engine malware-detection gateway written in C++ on Qt 6. Files are
submitted over an HTTPS REST API to an analysis broker that fans out to several
detection engines — signature scanning, binary disassembly, decompilation,
behavioral sandboxing, entropy/polymorphic heuristics, and AI-assisted code
context — and returns a single consensus verdict (BLOCK / QUARANTINE / ALLOW).

It also includes host-level network monitoring that feeds connection data to a
beaconing/C2 detector and behavioral-analytics engines.

> **Status: working prototype / research project.** The core detection pipeline
> is real and demonstrable (see [What actually works](#what-actually-works)).
> This is **not** a hardened production security product — see
> [Honest limitations](#honest-limitations) before relying on it for anything.

---

## What actually works

These are exercised end-to-end and verified by the included test suite on a
fresh Kali Linux install:

- **HTTPS REST API** with TLS and Bearer-token auth. Submit a file, get a JSON
  verdict.
- **Signature detection** — ClamAV + YARA. Catches known malware, including
  inside archives (ClamAV unpacks ZIP) and known packers (UPX).
- **Decode pre-pass** — base64/hex-encoded payloads are decoded and re-scanned,
  so a base64-wrapped known sample is still caught.
- **Binary analysis** — real disassembly (rizin + capstone) and decompilation
  (rz-ghidra) producing pseudo-C, plus entropy analysis that flags
  packed/encrypted content.
- **Behavioral sandbox** — files are detonated under `bwrap`/`strace` and scored
  on observed syscalls (memory-protection changes, network, persistence).
- **Multi-engine consensus** — all engines run on every file (no early exit);
  scores aggregate against a configurable threshold.
- **File-system watcher** — recursively monitors download directories and
  auto-submits files (by magic bytes, not just extension).
- **Connection monitoring** — polls `/proc/net/tcp` and feeds a beacon detector
  (periodicity/jitter/autocorrelation analysis) and behavioral-analytics engine.
- **Vulnerability feed** — live NVD / CISA KEV / GitHub Advisory ingestion.
- **IDS** — drives Suricata.
- **Native desktop GUI** — a Qt 6 control panel (`sentinel-gui`) that monitors
  and controls the firewall through its API (live status, detection counts,
  file submission, rules, vulnerabilities, log tail).

### Demo evidence

`sudo ./demo.sh` submits the standard EICAR test string and a benign file:

| Input | Verdict | Score | Engines |
|-------|---------|-------|---------|
| EICAR test file | **BLOCK** | 0.95 | AntiVirus, IDAEngine, DecompilerEngine, SandboxDetonator, PolymorphicAnalyzer, AICodeContextAnalyzer |
| Benign text file | **ALLOW** | 0.00 | (same, no signals) |

EICAR is a harmless industry-standard AV test string — not real malware.

---

## Quick start (fresh Kali Linux)

> Use a path with **no spaces or parentheses** (e.g. `~/SentinelV3`). The eBPF
> build step invokes clang via a shell and breaks on paths like
> `~/Downloads/files (2)/`.

```bash
# 1. Unpack into a clean path
unzip SentinelFirewall.zip -d ~/
cd ~/SentinelV3

# 2. Install (deps, build, deploy, TLS cert, config, systemd)
sudo ./install.sh

# 3. Start it (engines take ~35-45s to fully come online)
sudo ./run.sh

# 4. Run the live detection demo
sudo ./demo.sh

# 5. Run the honest test suite
sudo bash tests/hard_test.sh
```

`install.sh` is idempotent — safe to re-run. To rebuild only (skip dep install):
`sudo bash install.sh --build-only`.

---

## Architecture

```
   HTTPS request (TLS + Bearer auth)
        │
        ▼
   APIServer ───────────── hand-rolled HTTP/1.1 parser on a QTcpServer subclass
        │                  (raw-fd → single QSslSocket; auth, rate limit, audit log)
        ▼
   InlineAnalysisBroker ── queues + dispatches each request on a worker thread
        │
        ▼
   DownloadInspector ───── runs the engine pipeline (every engine, no short-circuit)
        │
        ├─► Decode pre-pass ────── base64/hex → re-scan decoded bytes
        ├─► AntiVirus ──────────── ClamAV + YARA signatures
        ├─► IDAEngine ──────────── rizin + capstone disassembly, entropy, structure
        ├─► DecompilerEngine ───── rz-ghidra pseudo-C, suspicious-API/pattern scan
        ├─► SandboxDetonator ───── bwrap + strace behavioral detonation
        ├─► PolymorphicAnalyzer ── entropy / packing heuristics
        └─► AICodeContextAnalyzer  optional LLM reasoning over decompiled code
        │
        ▼
   Consensus score vs block_threshold → BLOCK / QUARANTINE / ALLOW
        │
        ▼
   JSON response (TLS)  +  quarantine  +  forensic timeline record
```

A separate path — `ConnectionMonitor` → `BeaconDetector` / `UEBAEngine` /
`SSLInspector` — handles live network-behavioral detection by sampling
`/proc/net/tcp`.

---

## Detection types: signature vs heuristic vs behavioral

Be clear about which is which when interpreting results:

- **Signature (high confidence):** ClamAV/YARA matches. A hit is a strong
  positive. A miss means "no known signature," not "safe."
- **Heuristic (a signal, not proof):** entropy/polymorphic scoring. Flags
  packed/encrypted content — but legitimately compressed or encrypted files are
  also high-entropy, so this raises the score without blocking on its own. This
  is the firewall's edge over signature-only tools, with a measured
  false-positive tradeoff.
- **Behavioral (depends on what the sample does):** sandbox detonation and
  beacon detection observe behavior over time. A sample that doesn't act
  maliciously in the sandbox window, or hasn't beaconed long enough, won't be
  flagged yet.

---

## API

```bash
KEY=$(sudo grep -v '^#' /etc/sentinel/initial-api-key.txt | head -1)

# Health
curl -k https://127.0.0.1:8443/api/v1/health

# Submit a file for inline analysis
curl -k -X POST https://127.0.0.1:8443/api/v1/inline/submit \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d "{\"content_b64\":\"$(base64 -w0 somefile)\",\"filename\":\"somefile\"}"
```

For loopback-only / no-TLS operation: `PLAINTEXT=1 sudo ./run.sh`.

---

## Build requirements

Qt 6, OpenSSL 3.x, GCC/Clang (C++17), CMake + Ninja, and: `rizin`, `capstone`,
`unicorn`, `clamav`, `yara`, `suricata`, `bubblewrap`, `upx-ucl`. Optional:
`clang` + `gcc-multilib` for the eBPF/XDP component (auto-detected; build with
`-DWITH_EBPF=OFF` to skip). `install.sh` installs these on Debian/Kali.

Tested on Kali Linux (Qt 6.10, OpenSSL 3.6, GCC 15, rizin 0.8, ClamAV 1.4,
Suricata 8.0).

---

## Tests

```bash
sudo bash tests/hard_test.sh          # honest scorecard: hits, misses, FP rate, coverage
sudo bash tests/beacon_test.sh        # benign beacon-shape detection (takes ~3 min)
sudo bash tests/all_engines_smoke_test.sh
```

`hard_test.sh` deliberately reports **misses and false-positive behavior**, not
just successes — including the limits below. All test samples are harmless
(EICAR string, packed `/bin/true`, random bytes).

There are three test suites:

| Suite | What it checks |
|-------|----------------|
| `tests/hard_test.sh` | Core detection scorecard: signature hits, obfuscation, packing, false-positive discipline, watch-coverage boundary |
| `tests/obfuscation_test.sh` | Encoding/encryption resistance — base64/hex/XOR (caught) vs AES/RC4 (honestly flagged-not-decrypted) |
| `tests/beacon_test.sh` | Connection-based beacon detection (benign traffic with a beacon-like shape) |

---

## Walkthroughs

Two scenario documents show how the engines chain together:

- **[`demos/LIVE_DEMO_SCENARIO.md`](demos/LIVE_DEMO_SCENARIO.md)** — a runnable,
  step-by-step demo where every command produces a real, verifiable verdict.
- **[`demos/VISION_SCENARIO.md`](demos/VISION_SCENARIO.md)** — a reference-
  architecture narrative (an APT attack on an industrial site) with every
  capability tagged 🟢 operational / 🟡 partial / 🔵 roadmap, so the design
  vision is separated from what runs today.

---

## What I learned building this

This project taught me more than any course did. A few of the lessons, kept
honest:

- **The event loop is sacred.** My most recurring bug class was heavy,
  synchronous work (sandbox detonation, archive unpacking, a busy analysis loop)
  blocking Qt's event loop or starving a worker-thread pool — which made the API
  silently stop responding. I learned to push blocking work onto worker threads,
  bound how long any single operation can hold a thread, and treat "the server
  went quiet under load" as a concurrency smell, not a mystery.
- **Read the struct before you use it.** Several build failures came from me
  guessing a field name instead of opening the header. Cheap lesson, learned
  repeatedly until it stuck.
- **Honest detection has a ceiling, and that's fine.** You cannot read
  AES-encrypted bytes without the key — no tool can. I learned the difference
  between *obfuscation* (base64/hex/weak XOR — recoverable, and I do recover it)
  and *encryption* (real keys — correctly flagged as suspicious, not "detected").
  Claiming otherwise would be the easiest way to lose a reviewer's trust.
- **Layering beats any single engine.** No one technique catches everything.
  Signature misses the novel; entropy flags the benign-compressed; the sandbox
  misses the patient implant. The value is in combining independent signals — and
  in being explicit about what each one can and can't see.
- **Know your limits — write them down.** The strongest thing I can put in a
  security project isn't a feature list; it's an accurate account of where the
  tool stops. `AUDIT.md` and the limitations below exist for that reason.

If you work in this field and something here is naive or wrong, I'd value the
correction — that's the point of building in the open.

---

## Honest limitations

This project aims to be real engineering, so the gaps are documented rather than
hidden. See `AUDIT.md` for the full breakdown.

- **Not an inline network firewall (yet).** It analyzes files submitted to its
  API and watches download directories. It does **not** transparently intercept
  all network traffic — there is no NFQUEUE/proxy gateway. The "intercept every
  download in transit" and outbound-DLP features are designed in the scenario
  docs but **not implemented** (see `ROADMAP_LATER.md`).
- **The AI layer is optional and dormant by default.** Without an Anthropic API
  key it does nothing; the detection demo does not depend on it. When enabled,
  all "agents" are Claude with role-specific prompts (the orchestrator runs one
  model, not several).
- **Truly encrypted payloads can't be read statically.** No tool can decompile
  or read AES-encrypted bytes without the key. Such content is flagged as
  high-entropy/suspicious; the honest path to seeing inside is dynamic sandbox
  detonation, which only observes what the sample does in the sandbox window.
- **Beacon detection is connection-level and time-based.** It samples
  `/proc/net/tcp` (no packet payloads or microsecond timing — that needs
  pcap/AF_PACKET). It detects periodicity over minutes, and uses a CDN/cloud
  allowlist to reduce false positives — which means a C2 domain-fronted through
  a major CDN could be missed. This is a tradeoff, not a guarantee.
- **~62 engines are instantiated; ~22 are wired into cross-engine data flow.**
  Many engines run as standalone modules with real logic but are not part of an
  integrated correlation graph. Don't read the engine count as 62 integrated
  detectors.
- **Research/prototype quality.** Not audited for production deployment; expect
  rough edges. Licensed MIT (see `LICENSE`) — free to learn from, but **not**
  intended for production security use.

---

## Repository layout

| Path | What |
|------|------|
| `src/` | C++ sources (~46k LoC, 66 `.cpp` / 65 `.h`) |
| `gui/` | Native Qt 6 desktop control panel (`sentinel-gui`) |
| `tests/` | Three shell test suites + per-engine C++ tests |
| `demos/` | Live-demo and vision/reference-architecture walkthroughs |
| `ebpf/` | Optional XDP/eBPF kernel program |
| `python/` | ML training helpers (sklearn) |
| `dashboard/` | Static HTML status dashboard |
| `install.sh` / `run.sh` / `demo.sh` | Setup, run, and demo scripts |
| `AUDIT.md` | Honest per-engine engineering audit |
| `ROADMAP_LATER.md` | Designed-but-not-built features |
| `LICENSE` | MIT |

---

## Acknowledgements

Built on rizin, capstone, unicorn, ClamAV, YARA, Suricata, bubblewrap, and Qt 6.
EICAR test string courtesy of [eicar.org](https://www.eicar.org/).
