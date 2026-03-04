# 🛡️ Hypervisor Source Safety Database

> Community-driven malware analysis of hypervisor and game-cheat sources circulating on hacking forums.  
> **Goal:** Help users verify if source code they downloaded is safe before running or compiling it.

---

## ⚠️ Disclaimer

This project is for **educational and security research purposes only.**  
We do not distribute, link to, or endorse any software analyzed here.  
All analysis is performed on publicly circulating source code using static and dynamic analysis techniques.  
We are not affiliated with any forum, developer, or software vendor.

---

## 📊 Safety Database

| Source Name | Version / Build | Verdict | Date Analyzed | Report |
|---|---|---|---|---|
| AFOP.B1-HypervisorSources | B1 (2/25/2026) | ✅ SAFE | 2026-02-28 | [Full Report](reports/SAFE/AFOP.B1-HypervisorSources.md) |

---

## ✅ SAFE

### AFOP.B1-HypervisorSources

| Field | Details |
|---|---|
| **File** | `AFOP.B1-HypervisorSources.7z` |
| **Unpacked Size** | ~89.7 MB |
| **Date Analyzed** | February 28, 2026 |
| **Verdict** | ✅ **SAFE** |
| **Analyst** | Claude, Codex |

**Contents analyzed:**
- `HypervisorIntelSource/` — HyperDbg-based Intel hypervisor source
- `HypervisorAMDSource/` — SimpleSvm-based AMD hypervisor source

**Checks performed (Phase 0–8 Audit):**
- Static code analysis — all source files scanned (10 detection categories)
- Build script & supply chain review (`CMakeLists.txt`, `.vcxproj`, `.ps1`)
- Pre-compiled binary audit (`.obj`, `.lib` files verified against upstream Zydis/Zycore)
- Dependency integrity verification (Zydis, Keystone, ia32-doc, pdbex)
- Code provenance — matched against upstream HyperDbg and SimpleSvm repositories
- Scan for rootkit patterns (SSDT hooks, DKOM, hidden driver installation)
- Scan for hypervisor-specific threats (EPT weaponization, VMCALL backdoors, MSR abuse)
- Scan for process injection APIs (`CreateRemoteThread`, `VirtualAllocEx`, `WriteProcessMemory`)
- Scan for credential harvesting and keylogger routines
- Scan for persistence mechanisms (registry run keys, scheduled tasks, service install)
- Scan for data exfiltration (network calls, file system reads of sensitive paths)
- Scan for obfuscated payloads, shellcode arrays, and encoded blobs
- VM-exit handler review — all exit reason cases verified as legitimate
- Architecture verification — all suspicious hypervisor behaviors justified

**Findings:** No malicious, suspicious, or dangerous code patterns detected. Source code is consistent with a legitimate open-source hypervisor/debugger project. Codebase matches expected structure for HyperDbg and SimpleSvm derivatives.

---

## ❌ DANGEROUS / ⚠️ SUSPICIOUS

*No entries yet.*

---

## 🔬 Analysis Methodology

All sources are analyzed using a **comprehensive 8-phase audit framework**. See the full methodology: **[methodology.md](methodology.md)**

### Audit Phases:

| Phase | Name | Description |
|---|---|---|
| 0 | **Initial Triage** | Archive hashing, file inventory, binary artifact count, git history check |
| 1 | **Code Provenance** | Upstream identification, recursive diff against known projects, deviation mapping |
| 2 | **Build System & Supply Chain** | Build script audit, pre-compiled binary verification, dependency integrity |
| 3 | **Static Source Code Analysis** | 10-category automated pattern scan + manual deep review of critical functions |
| 4 | **Assembly Review** | Targeted review of `.asm` files for shellcode, raw syscalls, self-modifying code |
| 5 | **Resource & Data Audit** | `.rc`, `.inf`, `.reg`, `.dat`, `.bin` file inspection and entropy analysis |
| 6 | **Dynamic Analysis** | Sandboxed execution with Procmon, Wireshark, and WinDbg monitoring |
| 7 | **Architecture Verification** | Legitimacy assessment of inherently suspicious hypervisor behaviors |
| 8 | **Final Verdict** | 10-question checklist determining SAFE / SUSPICIOUS / DANGEROUS rating |

### Detection Categories (Phase 3):

| ID | Category | Severity |
|---|---|---|
| A | Process Injection APIs | CRITICAL |
| B | Credential Harvesting & Keylogging | CRITICAL |
| C | Rootkit / Kernel Hooks | HIGH |
| D | Hypervisor-Specific Threats (MSR, EPT, VMCALL) | HIGH |
| E | Interrupt & Exception Hijacking | HIGH |
| F | Anti-Analysis / Evasion | MEDIUM |
| G | Networking & Exfiltration | MEDIUM |
| H | Privilege Escalation | MEDIUM |
| I | Persistence Mechanisms | MEDIUM |
| J | Obfuscation & Encoded Payloads | HIGH |

### Tools used:
`ripgrep`, `Ghidra`, `x64dbg`, `WinDbg`, `Wireshark`, `Procmon`, `strings`, `7-Zip`, manual code review, PowerShell automated scanning

---

## 📝 How to Analyze a New Source

1. **Download and extract** the source archive in an isolated environment.
2. **Copy** [`reports/TEMPLATE.md`](reports/TEMPLATE.md) to `reports/SAFE/` or `reports/DANGEROUS/` with a descriptive filename.
3. **Follow the 8-phase methodology** in [`methodology.md`](methodology.md), filling in each section of the template.
4. **Determine the verdict** using the Phase 8 checklist.
5. **Add an entry** to the Safety Database table in this README.
6. **Commit and push** the report.

---

## 🤝 Contributing

Found a source you want analyzed? Open an issue with:
- The **name** of the source (no links)
- Where it is circulating (forum name only, no links)
- Any suspicious behavior you've noticed

**Do not post download links in issues or PRs.**

---

## 📁 Repo Structure

```
/
├── README.md              ← This file (main safety database)
├── methodology.md         ← Full 8-phase analysis methodology
├── reports/
│   ├── TEMPLATE.md        ← Report template for new analyses
│   ├── SAFE/
│   │   └── AFOP.B1-HypervisorSources.md
│   └── DANGEROUS/
│       └── (none yet)
└── [SOURCE_FOLDERS]/      ← Extracted source code under analysis
```

---

## 📄 License

This project (analysis reports and documentation) is licensed under [MIT License](LICENSE).  
We claim no ownership over any software analyzed.
