# 🛡️ Hypervisor Source Safety Database

> Community-driven malware analysis of hypervisor sources circulating on hacking forums.  
> **Goal:** Help users verify if source code they downloaded is safe before running or compiling it.

---

## ⚠️ Disclaimer

This project is for **educational and security research purposes only.**  
We do not distribute, link to, or endorse any software analyzed here.  
All analysis is performed on publicly circulating source code using static and dynamic analysis techniques.  
We are not affiliated with any forum, developer, or software vendor.

---

## 📊 Safety Database

| Source Name | Version / Build | Verdict | Date Analyzed | Notes |
|---|---|---|---|---|
| AFOP.B1-HypervisorSources | B1 (2/25/2026) | ✅ SAFE | 2026-02-28 | No malicious indicators found. See report below. |

---

## ✅ SAFE

### AFOP.B1-HypervisorSources

| Field | Details |
|---|---|
| **File** | `AFOP.B1-HypervisorSources.7z` |
| **Unpacked Size** | ~89.7 MB |
| **Date Analyzed** | February 28, 2026 |
| **Verdict** | ✅ **SAFE** |
| **Analyst** | Antigravity |

**Contents analyzed:**
- `HypervisorIntelSource/` — HyperDbg-based Intel hypervisor source
- `HypervisorAMDSource/` — SimpleSvm-based AMD hypervisor source

**Checks performed:**
- Static code analysis (all source files)
- Build script review (`CMakeLists.txt`, `.clang-format`)
- Dependency audit (`/dependencies`, `/libraries`)
- Scan for hardcoded IPs, C2 callbacks, obfuscated payloads
- Scan for rootkit patterns (SSDT hooks, DKOM, hidden driver installation)
- Scan for persistence mechanisms (registry run keys, scheduled tasks, service install)
- Scan for data exfiltration (network calls, file system reads of sensitive paths)
- Scan for credential harvesting or keylogger routines
- Scan for embedded binary blobs or shellcode in source

**Findings:** No malicious, suspicious, or dangerous code patterns detected. Source code is consistent with a legitimate open-source hypervisor/debugger project. Codebase matches expected structure for HyperDbg and SimpleSvm derivatives.

---

## ❌ DANGEROUS / ⚠️ SUSPICIOUS

*No entries yet.*

---

## 🔬 Analysis Methodology

All sources are analyzed using the following approach:

1. **Static analysis** — Full review of all source files for malicious patterns
2. **Build script audit** — Check for scripts that download or execute external content
3. **Dependency check** — Review third-party libraries for known malicious code
4. **Indicator scan** — Search for hardcoded IPs, URLs, obfuscated strings, shellcode
5. **Rootkit pattern scan** — Check for kernel hooks, DKOM, privilege escalation, persistence
6. **Network behavior review** — Identify any unexpected outbound communication logic

Tools used: Ghidra, x64dbg, Wireshark (dynamic), manual code review (static)

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
├── README.md          ← This file (main safety database)
├── reports/
│   ├── SAFE/
│   │   └── AFOP.B1-HypervisorSources.md   ← Detailed report
│   └── DANGEROUS/
│       └── (none yet)
└── methodology.md     ← Full analysis methodology
```

---

## 📄 License

This project (analysis reports and documentation) is licensed under [MIT License](LICENSE).  
We claim no ownership over any software analyzed.
