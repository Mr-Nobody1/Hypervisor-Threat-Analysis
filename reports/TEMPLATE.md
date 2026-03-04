# Analysis Report: [SOURCE_NAME]

## Metadata

| Field | Details |
|---|---|
| **File** | `[ARCHIVE_FILENAME]` |
| **SHA-256** | `[HASH]` |
| **Unpacked Size** | [SIZE] |
| **Date Analyzed** | [YYYY-MM-DD] |
| **Analyst** | [NAME] |
| **Verdict** | [✅ SAFE / ⚠️ SUSPICIOUS / ❌ DANGEROUS] |

---

## Phase 0 — Triage Summary

**Total files:** [N]
**File breakdown:**

| Extension | Count |
|---|---|
| `.c` / `.cpp` | |
| `.h` / `.hpp` | |
| `.asm` | |
| `.vcxproj` / `.sln` | |
| `.obj` / `.lib` | |
| `.ps1` / `.bat` / `.sh` | |
| Other | |

**Binary artifacts without source:** [N — list them or "None"]

**Git history present:** Yes / No
- If yes: [Number of commits, author names, any anomalies]

**Claimed purpose:** [What the README/docs say the project is]

---

## Phase 1 — Code Provenance

**Upstream project identified:** [Project name] / None

**Version / commit match:** [Tag/commit hash or "Could not determine"]

**Diff summary against upstream:**
- Files only in analyzed source (not in upstream): [list]
- Files modified from upstream: [list]
- Files only in upstream (missing from analyzed source): [list]

**Provenance risk:** [LOW — matches upstream / MEDIUM — partial match / HIGH — no upstream found]

---

## Phase 2 — Build System & Supply Chain

### Build scripts reviewed:

| File | Verdict | Notes |
|---|---|---|
| `[path]` | CLEAN / FLAG | [notes] |

### Pre-compiled binaries:

| File | Has Source? | Hash Match? | Verdict |
|---|---|---|---|
| `[path]` | Yes / No | Yes / No / N/A | CLEAN / FLAG |

### Dependencies:

| Dependency | Claimed Source | Version Verified? | Modified? | Verdict |
|---|---|---|---|---|
| `[name]` | [upstream URL] | Yes / No | Yes / No | CLEAN / FLAG |

---

## Phase 3 — Static Source Code Analysis

### Category A: Process Injection APIs (CRITICAL)
- [ ] Scanned — **Findings:** [None / list]

### Category B: Credential Harvesting & Keylogging (CRITICAL)
- [ ] Scanned — **Findings:** [None / list]

### Category C: Rootkit / Kernel Hooks (HIGH)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category D: Hypervisor-Specific Threats (HIGH)
- [ ] Scanned — **Findings:** [None / list with justification]

**VM-Exit Handler Review:**

| Exit Reason | Handler Location | Behavior | Verdict |
|---|---|---|---|
| CPUID | `[file:line]` | [description] | CLEAN / FLAG |
| MSR Read/Write | `[file:line]` | [description] | CLEAN / FLAG |
| EPT Violation | `[file:line]` | [description] | CLEAN / FLAG |
| VMCALL / VMMCALL | `[file:line]` | [description] | CLEAN / FLAG |
| [Other] | `[file:line]` | [description] | CLEAN / FLAG |

### Category E: Interrupt & Exception Hijacking (HIGH)
- [ ] Scanned — **Findings:** [None / list]

### Category F: Anti-Analysis / Evasion (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category G: Networking & Exfiltration (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category H: Privilege Escalation (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category I: Persistence Mechanisms (MEDIUM)
- [ ] Scanned — **Findings:** [None / list with justification]

### Category J: Obfuscation & Encoded Payloads (HIGH)
- [ ] Scanned — **Findings:** [None / list]

---

## Phase 4 — Assembly Review

| File | Key Instructions Found | Verdict | Notes |
|---|---|---|---|
| `[path]` | [instructions] | CLEAN / FLAG | [notes] |

---

## Phase 5 — Resource & Data Files

| File | Type | Contents/Purpose | Verdict |
|---|---|---|---|
| `[path]` | [type] | [description] | CLEAN / FLAG |

---

## Phase 6 — Dynamic Analysis

**Performed:** Yes / No (reason: [if no, explain why])

### Observations:
- **File system activity:** [None / list]
- **Registry activity:** [None / list]
- **Network activity:** [None / list]
- **Process creation:** [None / list]

---

## Phase 7 — Architecture Verification

| Suspicious Behavior | File(s) | Legitimate Justification | Verdict |
|---|---|---|---|
| [behavior] | `[path:line]` | [justification or "None"] | CLEAN / FLAG |

---

## Phase 8 — Final Verdict

### Verdict Checklist

| # | Question | Answer |
|---|---|---|
| 1 | Process injection APIs used against external processes? | YES / NO |
| 2 | Credential harvesting or keylogging functionality? | YES / NO |
| 3 | Outbound network connections to non-documentation URLs? | YES / NO |
| 4 | Persistence beyond own driver loading? | YES / NO |
| 5 | Functionality deviating from stated purpose? | YES / NO |
| 6 | Undocumented VMCALL/backdoor interfaces? | YES / NO |
| 7 | EPT/NPT weaponization beyond debugging? | YES / NO |
| 8 | Downloads/executes external content during build or runtime? | YES / NO |
| 9 | Pre-compiled binaries without corresponding source? | YES / NO |
| 10 | Obfuscated payloads, shellcode, or high-entropy embedded data? | YES / NO |

### Overall Verdict: [✅ SAFE / ⚠️ SUSPICIOUS / ❌ DANGEROUS]

### Summary
[2-4 sentence summary explaining the verdict. Reference the most significant findings or lack thereof.]

---

## High / Critical Findings Summary

| # | File | Line(s) | Category | Severity | Finding | Verdict |
|---|---|---|---|---|---|---|
| 1 | | | | | | |

*If no HIGH/CRITICAL findings, write "No high or critical findings detected."*

---

## Recommended Actions
- [ ] [Action item 1, e.g., "Compare X file against upstream commit Y"]
- [ ] [Action item 2]
