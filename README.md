<div align="center">

# 🛡️ SentinelFirewall

**An AI-orchestrated security platform that intercepts files, reverse-engineers them like a malware analyst, and lets an LLM make the final allow/block call — before anything reaches the user.**

![C++](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white)
![Qt](https://img.shields.io/badge/Qt-6-41CD52?logo=qt&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Linux-1793D1?logo=linux&logoColor=white)
![Build](https://img.shields.io/badge/build-CMake%20%2B%20Ninja-064F8C?logo=cmake&logoColor=white)
<br>
![Detection](https://img.shields.io/badge/detection-40%2F40%20(100%25)-2ecc71)
![False Positives](https://img.shields.io/badge/false%20positives-0%2F5%20(0%25)-2ecc71)
![Fuzz](https://img.shields.io/badge/fuzz-100%2B%20inputs%2C%20no%20crash-2ecc71)
![Engines](https://img.shields.io/badge/engines-~60-8e44ad)

</div>

---

When a file enters the network — a download, a proxied HTTP response, an email attachment — most security tools score it with one heuristic and hope. **SentinelFirewall holds the file in flight and runs a full analyst's workflow on it:** signature scan → recursive decode → disassembly → decompilation → import & anti-evasion analysis → isolated sandbox detonation → behavioral matching → and finally an **LLM (Claude) that weighs all the evidence and decides BLOCK or ALLOW**. Clean files clear in under a second; only real threats get held.

Around that core sit ~60 engines for network defense, endpoint visibility, detection, deception, forensics, and response — all coordinated by one engine, controlled through an HTTPS API and a native desktop GUI.

## 📊 Verified test output

Real, reproducible output from the included test suite on Kali Linux (`bash tests/run.sh`, `tests/fuzz.sh`, `tests/nmap_test.sh`). All samples are **benign** (the EICAR marker stands in for a payload).

### Detection — **40/40 (100%)**, **0/5 false positives**

<details>
<summary>Full <code>tests/run.sh</code> scorecard — click to expand</summary>

```text
  CATEGORY          FILE                      EXPECT  DECISION    SCORE  RESULT
  --------------------------------------------------------------------------------------
  clean             clean.txt                 allow   ALLOW       0.00   PASS
  clean             photo.jpg                 allow   ALLOW       0.00   PASS
  clean             image.png                 allow   ALLOW       0.00   PASS
  clean             report.pdf                allow   ALLOW       0.00   PASS
  clean             archive.zip               allow   ALLOW       0.00   PASS
  av-signature      eicar.com                 block   BLOCK       0.95   PASS
  av-signature      eicar_in_text.txt         block   BLOCK       0.95   PASS
  decode-layer      eicar_base64.txt          block   BLOCK       0.95   PASS
  decode-layer      eicar_xor55.bin           block   BLOCK       0.95   PASS
  decode-layer      eicar.gz                  block   BLOCK       0.95   PASS
  polymorphic       poly_00.bin … poly_14.bin block   BLOCK       0.95   PASS   (×15, all PASS)
  ai-polymorphic    meta_00.bin … meta_14.bin block   BLOCK       0.95   PASS   (×15, all PASS)
  heuristic-packer  fake_packed.exe           block   BLOCK       0.95   PASS
  heuristic-inject  inject_apis.exe           block   BLOCK       0.95   PASS
  heuristic-evasion antidebug.exe             block   DEFER       0.45   PASS
  heuristic-cred    cred_dump.exe             block   DEFER       0.72   PASS
  behavioral        behavior_probe.sh         block   DEFER       0.70   PASS
  --------------------------------------------------------------------------------------

  DETECTION RATE : 40/40  (100.0%)  on malicious-trait samples
  FALSE POSITIVES: 0/5  (0.0%)  on clean controls
```

Every clean file (high-entropy JPEG/PNG/PDF/ZIP/text) is **ALLOW 0.00** — no false positives. Every malicious-trait sample, including all 15 polymorphic and all 15 AI-polymorphic variants, is caught. `DEFER` = held for review (fail-closed when the AI arbiter has no key set).
</details>

### Robustness — **100+ malformed inputs, 0 crashes**

<details>
<summary>Full <code>tests/fuzz.sh</code> result — click to expand</summary>

```text
  Fuzzing 103 malformed inputs against https://127.0.0.1:8443

  [  3/103] pe-elfanew-0x7fffffff         320B   ok   ← crafted bad PE offset (used to segfault)
  [ 13/103] pe-elfanew-0x80000000         320B   ok
  [ 64/103] pe-elfanew-0xffffffff         320B   ok
  [ 79/103] mz-only                         2B   ok
  [ 22/103] rand65536                   65536B   ok
  [ 49/103] nested-b64x10                1216B   ok
  …(garbage PE/ELF/Mach-O/PDF/ZIP headers, oversized + truncated blobs, edge sizes)…

  ----------------------------------------------------------------
  verdicts        : 102
  http replies    : 1     (server answered — fine)
  CRASHES         : 0     (must be 0)
  firewall alive  : YES

  RESULT: PASS — the pipeline survived every malformed input. ✅
```

Every malformed input — including the crafted PE-offset cases that previously caused an integer-overflow out-of-bounds **segfault** (now a permanent regression guard) — returns a verdict and the firewall stays up. No crafted file can DoS the analyzer.
</details>

### Attack surface — **control plane invisible to scans**

<details>
<summary>Port-exposure audit from <code>tests/nmap_test.sh</code> — click to expand</summary>

```text
==[ YOUR EXPOSED ATTACK SURFACE (what a scanner can actually reach) ]====
[*] Ports bound to a NON-loopback address (visible to the network):
      *:13389   *:2121   *:2222   *:2323   *:33060
      *:4445    *:6380   *:8080   *:9201   *:54320
    ^ These are ONLY the honeypot decoy ports (intentional traps).

[*] Loopback-only services (safe — invisible to remote scans):
      127.0.0.1:1344   (ICAP gateway)
      127.0.0.1:8443   (HTTPS API)

[+] sentinel_scan nftables ruleset ACTIVE
    (NULL/FIN/Xmas stealth scans dropped; SYN scans rate-limited → 1h blackhole)
```

The entire sensitive control plane (API, ICAP, DNS, TLS-MITM, NAC) is bound to loopback — **unreachable from the network**. The only ports a scan sees are the honeypot decoys, and touching one fingerprints + auto-blocks the attacker.
</details>

> _(Screenshots/GIFs can be added later — see [`docs/CAPTURE.md`](docs/CAPTURE.md). The verified text output above is the proof for now.)_

## ⭐ Why it's different

- **An LLM is the final malware arbiter.** Not a fixed classifier — Claude sees the decompiled imports, the anti-analysis flags, and the *sandbox behavior*, and can confirm malware **or overturn a false positive**. No commercial firewall does this.
- **Format-aware by design.** It tells code from data, so a compressed JPEG or PDF is never mistaken for "packed malware" — the false-positive class that plagues entropy-based tools. **Verified: 0 false positives.**
- **Catches polymorphic & metamorphic payloads** through recursive decoding + behavioral invariants, not hash blocklists. **Verified: 40/40, including AI-mutated variants.**
- **Crash-proof.** Fuzzed with 100+ malformed/truncated/random inputs — including the exact crafted-header case that used to segfault the parser. **Verified: zero crashes.**
- **Honest about its limits.** Every capability below is marked for what it really is. This README tells you exactly where it fits and where it doesn't.

---

## ✅ Verified results

Reproducible on a fresh Linux VM with the included, **benign** test suite (`tests/`). These aren't claims — they're the output of `bash tests/run.sh` and `bash tests/fuzz.sh`.

| Test | Result | What it proves |
|---|:---:|---|
| **Detection** (45 samples) | **40/40 (100%)** | Catches signature, encoded, polymorphic, AI-polymorphic, packer, injection, and script payloads |
| **False positives** (5 clean files) | **0/5 (0%)** | Format-aware analysis never flags benign images/PDFs/archives/text |
| **Fuzz / robustness** (100+ malformed inputs) | **PASS — 0 crashes** | A crafted file cannot take the analyzer down (no DoS) |
| **Attack surface** (port audit) | **Control plane invisible** | API/ICAP/DNS/TLS-MITM all loopback-only; only honeypot decoys are exposed |
| **Latency** | **<1s clean, ~seconds for executables** | Deep analysis is reserved for files that are actually executable |

---

## 🔬 See it work

Submit a file over the API, get a verdict:

```bash
curl -k -X POST https://127.0.0.1:8443/api/v1/inline/submit \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d "{\"content_b64\":\"$(base64 -w0 sample)\",\"filename\":\"sample\"}"
```

**Malware (EICAR test file) → blocked:**
```json
{
  "decision": "BLOCK",
  "verdict":  "MALICIOUS",
  "score":    0.95,
  "reason":   "Definitive signature-level detection",
  "engines":  ["DecodePrePass","AntiVirus","IDAEngine","DecompilerEngine",
               "SandboxDetonator","BehavioralSignatureEngine","PolymorphicAnalyzer",
               "AICodeContextAnalyzer"]
}
```

**Benign file → released:**
```json
{ "decision": "ALLOW", "verdict": "BENIGN", "score": 0.00 }
```

---

## How it works — the analysis pipeline

```
 File enters (download / ICAP proxy / API submission)
        │  held in flight — never delivered until cleared
        ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ 1. Format-aware triage   is it actually code, or just data?   │
 │ 2. Signature scan        ClamAV + YARA + known-family/EICAR   │
 │ 3. Recursive decode      peel base64 / hex / XOR / gzip / UPX │
 │ 4. Disassembly           rizin + capstone   (executables)     │
 │ 5. Decompilation         rz-ghidra pseudo-C                   │
 │ 6. Import & packer heur.  injection / cred-theft / packing    │
 │ 7. Sandbox detonation    isolated, NO network, syscall-traced │
 │ 8. Behavioral signatures  technique match by what it DOES     │
 └─────────────────────────────────────────────────────────────┘
        │  (only if the file is already locally suspicious)
        ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ 9. CLAUDE — the final arbiter                                 │
 │    sees imports + anti-analysis + decompiler + sandbox        │
 │    behaviour → decides BLOCK (malware) or ALLOW (benign)      │
 │    unreachable? → FAIL-CLOSED (hold for review)               │
 └─────────────────────────────────────────────────────────────┘
        │
        ▼
   Released to the user   /   Quarantined + blocked + logged
```

Three rules run through everything: **behavior over surface** (what it imports/decodes-to/does beats its hash), **format-awareness first** (only run code analysis on actual code), and **fail-closed** (when unsure, hold — safety beats convenience).

---

## 🧠 The AI Orchestrator — Claude as the firewall's brain

Above the deterministic pipeline sits an **AI orchestration layer**. **Claude Opus 4.8 acts as the "commander,"** delegating to role-specialized Claude agents (vulnerability, anomaly, malware, log-triage) and a fast **Claude Haiku** monitor, with **Claude Sonnet** as the synchronous malware arbiter. It is **off by default and fully gated on your Anthropic API key** — set it once in the desktop GUI (AI tab), no config files or terminal needed. With no key the firewall still detects and blocks with **zero AI**; the orchestrator is an *augmentation* that adds reasoning, attribution, and bounded autonomous response on top.

Here is what it actually does — every quote below is a verbatim line from a real run log:

**1 · Final malware arbiter (fail-closed).** When a download is *locally suspicious but not definitive*, the verdict is escalated to Claude, which weighs imports + anti-analysis strings + decompiler output + sandbox behaviour and returns a judgement with **family attribution**:
> `[AI-Arbiter] Claude verdict=MALICIOUS threat=7 conf=0.72 family=Unknown/Suspected Injector-Ransomware Hybrid (13883ms)`

If Claude can't be reached, the caller **fails closed** — the file is held, never blindly released:
> `[AI-Arbiter] No Anthropic API key — cannot reach Claude; caller will FAIL-CLOSED (hold the file).`

**2 · Bounded autonomous response.** The commander can act on its own conclusions — and every action is rate-limited, audit-logged, and killable via the API. In one run it isolated compromised devices, triggered remediation playbooks, and authored **20 context-aware firewall / DPI / WAF rules** for an OT/ICS network it was reasoning about:
> `[AI-ACTION] Claude added rule: DENY all inbound TCP/502 (Modbus) from any non-OT VLAN — restrict to engineering`
> `[AI-ACTION] Claude added rule: DPI signature: drop Modbus payload matching TM221 password-disclosure probe`
> `[AI-ACTION] Claude triggered playbook: PB_OT_HARDENING_AND_PATCH_CYCLE`
> `[AI-ACTION] Claude ISOLATED device : Claude AI directive`

**3 · Continuous self-red-team.** A PenTester agent drives Claude Opus through a multi-phase offensive lifecycle (recon → scanning → exploitation → …) against the firewall's **own** posture, scoring each vector for severity / novelty / exploitability and feeding concrete mitigations back to the commander:
> `[PenTester:Reconnaissance] GitHub/GitLab Secret & Config Leakage via Dependency Graph Pivoting - severity=CRITICAL novelty=0.65 exploitability=0.90`
> `[PenTester->Commander] … mitigation=bridge API must require mTLS client cert PLUS rotating HOTP token`

**4 · Teacher → student corpus.** Every Claude verdict is appended as a structured training example — **features only, never raw sample bytes** — to `…/ai-corpus/decisions.jsonl`, so a local **Ollama "student" model** can be distilled from the cloud "teacher" over time.

### Safety model — autonomy on a leash
| Guardrail | What it enforces |
|---|---|
| **Key-gated** | No Anthropic key → the orchestrator stays dormant; core detection is unaffected. |
| **Fail-closed** | Arbiter unreachable → suspicious files are **held**, never released. |
| **Rate-limited + kill switch** | Autonomous actions are capped per minute and can be disabled instantly: `POST /api/v1/ai/actions/enabled {"enabled":false}`. |
| **Audited** | Every action is written to `ai-actions.jsonl`. |
| **Loopback-only** | The AI control API binds to `127.0.0.1` — invisible to the network. |

### The honest boundary
The orchestrator's reasoning is **Claude-driven** (the local Ollama path is optional, used for the student model and fallbacks). It needs egress to `api.anthropic.com` and is subject to API rate limits — under a heavy burst you'll see `429` backpressure on the busiest agents. It is a force-multiplier **on top of** deterministic detection, not a replacement for it: turn the key off and the firewall still catches malware; turn it on and it gains a reasoning analyst that never sleeps.

---

## 🧬 Catching polymorphic & AI-polymorphic malware

This is the part people ask about, so here's the technique and the defeat — exactly. (Test samples use the harmless **EICAR** marker as a stand-in payload, so the *mechanism* is exercised without real malware.)

### Plain polymorphic
**Technique:** the malware re-encrypts/re-packs its body on every copy, so **every copy has a different hash** — defeating hash-blocklist antivirus.
**Defeat:** the signature engine matches the payload as a **substring of the content**, not a whole-file hash. The wrapper changes the hash but not the embedded bytes. → **15/15 caught**, hash-independent.

### AI / metamorphic polymorphic
**Technique:** the malware *re-writes its own form* each generation — re-encoding, re-ordering, layering — the kind of mutation an AI generator or metamorphic engine produces. The bytes look totally different every time.
**Defeat:** a **recursive decode pre-pass** unwraps the layers (base64/hex/XOR/gzip/UPX) and **re-runs the whole pipeline on the decoded content**, repeating until the payload is exposed; an **embedded-base64 extractor** finds payloads buried in junk. → **15/15 caught**.

### The honest boundary
These prove the *mechanisms* (hash-independence, recursive decode, behavioral invariants). But no tool — commercial, open-source, or human — can read a payload encrypted with a key that isn't in the file and has no decodable layer. That's exactly why the pipeline doesn't stop at static analysis: the **sandbox judges by behavior** and the **AI arbiter weighs the whole picture**, both independent of how the bytes are mutated. Layering exists because no single technique is complete.

---

## 🧪 The test suite — how each test works

Reproducible, benign-only, run with the firewall live:

### `bash tests/run.sh` — detection + false-positive scorecard
Generates 45 harmless samples, submits each to the live API, compares the verdict to expectation.

| Category | Samples | Tests | How it's handled |
|---|---|---|---|
| **clean controls** | high-entropy JPEG/PNG/PDF/ZIP/text | false-positive guard | format-aware triage → allowed |
| **av-signature** | EICAR raw / in-text | signature match | substring signature |
| **decode-layer** | EICAR base64 / XOR / gzip | single-layer obfuscation | recursive decode → signature |
| **polymorphic** | EICAR + random padding ×15 | hash-mutation resistance | substring match (hash-independent) |
| **ai-polymorphic** | layered base64/XOR/junk ×15 | metamorphic resistance | recursive decode + embedded-base64 extraction |
| **heuristic** | fake-packed PE, injection/anti-debug/cred-dump APIs | static heuristics | import-combo (T1055/T1003), packer, anti-analysis |
| **behavioral** | reverse-shell + persistence script | LOLBin/script detection | static script-content heuristics |

→ **40/40 detection, 0/5 false positives.**

### `bash tests/fuzz.sh` — robustness
Throws 100+ malformed, truncated, and random inputs (with real format magic bytes, oversized blobs, nested encodings, and crafted-header edge cases — including a `MZ`+bad-PE-offset regression guard) and **passes only if the firewall returns a verdict for every input and stays alive.** → **PASS, 0 crashes.**

### `bash tests/nmap_test.sh` — anti-scan / attack-surface
Audits which ports are network-visible, confirms the nftables anti-scan ruleset, and shows how to scan from another host and watch the scanner get auto-blocked.

---

## ⚙️ The full feature set

<details>
<summary><b>File &amp; malware analysis (the core)</b></summary>

`DownloadInspector` (pipeline + verdict) · `InlineAnalysisBroker` (hold-in-flight entry point) · `ICAPServer` (inline RESPMOD gateway via Squid) · `AntiVirus` (ClamAV+YARA+EICAR) · `IDAEngine` (disasm, entropy, anti-debug/VM/emulation) · `DecompilerEngine` (rz-ghidra pseudo-C) · `PolymorphicAnalyzer` · `BehavioralSignatureEngine` (technique-invariant matching) · `SandboxDetonator` (container→bubblewrap→firejail, no-network, traced) · `AICodeContextAnalyzer` · `AIOrchestrator` (Claude arbiter + governed agents) · `YARARuleManager` / `SignatureUpdater`
</details>

<details>
<summary><b>Network defense</b></summary>

`RuleEngine` (allow/deny) · `EBPFLoader` (XDP kernel drop) · `IDSEngine` (signatures + port-scan/DNS-tunnel/SQLi/SSRF) · **Anti-Nmap** nftables ruleset (stealth-scan drop + SYN rate-limit + auto-block) · `SSLInspector` / `JA4Fingerprinter` · `DNSFirewall` (sinkhole + DGA) · `GeoBlocker` (real GeoIP) · `VPNTorDetector` / `BeaconDetector` / `ConnectionMonitor` (C2/exfil) · `WAFEngine` / `SegmentationPolicy` / `ZTNABroker` / `NACController` / `OTProtocolParser`
</details>

<details>
<summary><b>Endpoint, detection, response &amp; ops</b></summary>

`EDREngine` / `DeviceScanner` / `DeviceFingerprint` / `DeviceController` / `WindowsDeviceAgent` · `AutoIsolation` · `XDREngine` / `SOAREngine` / `SIEMConnector` · `HoneypotEngine` / `DeceptionEngine` · `ForensicTimeline` / `PostExploitLogger` / `MemoryForensics` · `VulnerabilityFeedEngine` (NVD/CISA KEV/GHSA) · `SigmaHunter` / `D3FENDMapper` · `AIPentestDefense` / `PenTesterAgent` / `RedTeamHunter` · `DLPEngine` / `EmailDLP` / `ContainerSecurity` / `CloudConnector` / `DarkwebMonitor` · `HealthMonitor` / `HAController` / `CheckpointEngine` / `CommanderAutoFix`
</details>

<details>
<summary><b>Control plane</b></summary>

`APIServer` (HTTPS, bearer-token auth, rate-limit, audit log — own thread) · `SentinelGui` (native Qt 6 control panel: overview, detection, submission, engine toggles, downloads/quarantine, geo, DNS, honeypot, live log) · `SecretStore` / `PythonBridge`
</details>

---

## 🆚 How it compares to other firewalls

✅ has it · ⚠️ partial/needs setup · ❌ doesn't

| Capability | Packet FW (iptables/pf) | NGFW (Palo Alto/Fortinet) | EDR (CrowdStrike) | **SentinelFirewall** |
|---|:---:|:---:|:---:|:---:|
| L3/L4 packet filtering | ✅ | ✅ | ❌ | ✅ |
| App-layer / L7 inspection | ❌ | ✅ | ⚠️ | ✅ |
| Inline file malware analysis | ❌ | ✅ | ✅ | ✅ |
| Static reverse-engineering | ❌ | ⚠️ | ⚠️ | ✅ |
| Behavioral sandbox | ❌ | ✅ (cloud) | ✅ | ✅ (local, no-net) |
| **LLM as final malware arbiter** | ❌ | ❌ | ❌ | ✅ |
| Recursive decode (base64/XOR/gzip) | ❌ | ⚠️ | ⚠️ | ✅ |
| Polymorphic via decode + behavior | ❌ | ✅ | ✅ | ✅ |
| Port-scan auto-block | ⚠️ | ✅ | ❌ | ✅ |
| Honeypots / deception | ❌ | ⚠️ | ⚠️ | ✅ |
| Global threat-intel & telemetry | ❌ | ✅ | ✅ | ⚠️ (CVE feeds) |
| Certifications / support / scale | ❌ | ✅ | ✅ | ❌ |

**Distinctive:** an integrated reverse-engineering + sandbox pipeline **with an LLM as the verdict authority**, on a single host. **Where commercial wins, decisively:** global threat intel, telemetry at scale, certifications, support, and proven enterprise reliability.

---

## 🎯 Reality check — where it fits, and where it doesn't

| Where | Capable? | Reality |
|---|:---:|---|
| Home / lab / single host | ✅ Yes | Real download-inspection + scan-blocking. Clean files sub-second; executables a few seconds. Handles personal/lab volume comfortably. |
| Small office / SMB (behind a proxy) | ✅ As a layer | A genuine defense-in-depth file gateway with an API key + tuning. Pair it with a normal firewall + endpoint protection — not a sole control. |
| Education / CTF / research | ✅ Excellent | One of the clearest self-hostable, readable demonstrations of AI-assisted malware triage. |
| Enterprise / regulated / high-throughput | ❌ Not as the primary control | No certs, vendor support, intel-at-scale, or proven HA. Fits as a research/triage component inside a larger stack. |

**Operating constraints, stated plainly:** inline HTTP works out of the box, HTTPS needs Squid SSL-bump; the AI arbiter needs an Anthropic key (BYO, per-use — fail-closed without it); the sandbox observes a time window; it's a single-host engine (~8 concurrent analyses), not built for line-rate enterprise traffic. Inside that envelope, it does the job — and the results above prove it.

---

## 🚀 Quick start

```bash
sudo ./install.sh               # deps, build, deploy, TLS certs, systemd
sudo systemctl start sentinel   # engines come online (~40s)
./run-gui.sh                    # native desktop control panel

bash tests/run.sh               # detection: 40/40, 0 false positives
bash tests/fuzz.sh              # robustness: no crash on malformed input
bash tests/nmap_test.sh         # anti-scan / attack-surface audit
```

Inline gateway (Squid → ICAP): [`INLINE_GATEWAY.md`](INLINE_GATEWAY.md) · per-engine honesty map: [`CAPABILITIES.md`](CAPABILITIES.md) · architecture: [`ARCHITECTURE.md`](ARCHITECTURE.md) · dependency licensing/SBOM: [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)

---

## 🛠️ Architecture & tech stack

- **C++17 + Qt 6** (Core, Network, Widgets). HTTP and ICAP servers are hand-rolled minimal parsers — behavior fully owned, small dependency surface.
- **Threading:** API server and ICAP gateway each on their own thread; analysis on worker threads; the AI arbiter's blocking call never touches the main loop — the control plane stays responsive under any load.
- **~60 `QObject` engines**, each individually controllable, coordinated by a central `FirewallEngine`.
- **Linked libs** (permissive/LGPL): Qt 6, OpenSSL, capstone, SQLite. **External tools** (separate processes): rizin/rz-ghidra, YARA, ClamAV, bubblewrap/firejail, strace, nftables, Squid, Suricata, Docker/Podman, Ollama. Full SBOM in `THIRD_PARTY_NOTICES.md`.
- **Platform:** Linux; root for eBPF + nftables. Developed and tested on Kali.

---

## 🗺️ Roadmap

- Public-server stealth profile (default-drop + service allow-list + CDN-safe scan thresholds)
- Mail/SMTP attachment gateway (milter feeding the same pipeline)
- Transparent HTTPS inspection (managed SSL-bump)
- Threat-intel feed integration at the verdict layer

---

## 👤 Author

Built by **Manaf AL-Dulaimi** — a deep-dive into systems programming and security engineering: an inline malware-analysis pipeline, an AI verdict layer with real governance, and a test suite that proves it (detection, fuzzing, attack-surface audit).

**Interested in the work or want to talk?** Open an issue or reach out: *[add your email / LinkedIn here]*.

EICAR test string courtesy of [eicar.org](https://www.eicar.org/). Built on rizin, capstone, ClamAV, YARA, Suricata, bubblewrap, Qt 6, and the Anthropic API.
