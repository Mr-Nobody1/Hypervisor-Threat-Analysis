# 🔬 Analysis Methodology

> Comprehensive, repeatable audit framework for hypervisor and game-cheat source code circulating on hacking forums.
> This document serves as the **canonical template** — every new source analysis MUST follow these steps in order.

---

## How to Use This Template

When analyzing a new hypervisor or game-cheat source package:

1. **Create a new report** in `reports/SAFE/` or `reports/DANGEROUS/` using `reports/TEMPLATE.md` as the skeleton.
2. **Follow each phase below in order** — do not skip phases.
3. **Record every finding** with file path, line number(s), code snippet, and your verdict (CLEAN / LOW / MEDIUM / HIGH / CRITICAL).
4. **Answer the final verdict checklist** at the bottom of the template to determine the overall safety rating.

---

## Phase 0 — Initial Triage & Metadata Collection

Before reading any code, establish the basics:

| Check | Details |
|---|---|
| **Archive hash** | Compute SHA-256 of the original archive (`.7z`, `.zip`, `.rar`) before extraction. |
| **Unpacked size** | Record total size on disk after extraction. |
| **File inventory** | Generate a complete file listing with extensions. Use `find . -type f` or `Get-ChildItem -Recurse`. |
| **Extension census** | Count files by extension (`.c`, `.cpp`, `.h`, `.asm`, `.obj`, `.lib`, `.sys`, `.dll`, `.exe`, `.ps1`, `.bat`, `.py`, `.rc`). |
| **Binary artifact count** | How many pre-compiled binaries (`.obj`, `.lib`, `.sys`, `.dll`, `.exe`, `.bin`) exist? Flag any that don't have corresponding source. |
| **Git history** | If `.git/` exists: check `git log --oneline -20`, author names/emails, force-push history, stashed changes (`git stash list`), unreachable objects (`git fsck --unreachable`). |
| **README / documentation** | What does the project claim to be? Note the stated purpose. |

**Output:** A metadata block at the top of the report including archive hash, file counts, and claimed purpose.

---

## Phase 1 — Code Provenance & Upstream Verification

Determine if the code is original or derived from a known project.

### 1.1 — Identify the upstream project
- Search for copyright headers, author names, license files, repository URLs in comments.
- Common upstream hypervisors to check against:
  - **HyperDbg** (Intel VT-x debugger)
  - **SimpleSvm / SimpleVisor** (AMD-V / Intel minimal hypervisors by Satoshi Tanda / Alex Ionescu)
  - **hvpp** (Petr Beneš)
  - **HyperBone** (DarthTon)
  - **DdiMon** (Satoshi Tanda)
  - **AetherVisor** (MellowNight)
  - **Shark** (FLAVOR)

### 1.2 — Diff against upstream
If an upstream is identified:
- Clone the upstream at the matching version/tag.
- Run a recursive diff: `diff -rq ./source ./upstream --exclude=.git`
- **Every file that exists in the analyzed source but NOT in upstream is a priority target** — this is where malicious modifications would hide.
- **Every modified file** must be diffed line-by-line. Document all deviations.

### 1.3 — Unknown provenance
If no upstream match can be found:
- Flag the source as **HIGHER RISK** — the code cannot be cross-referenced.
- All files must be manually reviewed with extra scrutiny.

**Output:** Provenance section listing upstream project (if any), version/commit, and a summary of all deviations.

---

## Phase 2 — Build System & Supply Chain Audit

Examine every file involved in compiling the project. Build scripts are the most common vector for supply-chain attacks.

### 2.1 — Build script inventory
Audit every file with these extensions: `.sln`, `.vcxproj`, `.vcxproj.filters`, `.props`, `.targets`, `.cmake`, `CMakeLists.txt`, `Makefile`, `.ps1`, `.bat`, `.cmd`, `.sh`, `.py`

### 2.2 — Checks for each build file

| What to look for | Why it matters |
|---|---|
| `Invoke-WebRequest`, `curl`, `wget`, `DownloadFile`, `URLDownloadToFile`, `git clone` | External download during build = supply-chain attack vector |
| `FetchContent`, `ExternalProject_Add`, `execute_process` (CMake) | CMake can fetch and run arbitrary code at configure time |
| `PreBuildEvent`, `PostBuildEvent`, `CustomBuild` (in `.vcxproj`) | Visual Studio can run arbitrary commands before/after compilation |
| `#pragma comment(linker, ...)` in source code | Can silently link unexpected libraries |
| Compiler flags: `/SAFESEH:NO`, `/DYNAMICBASE:NO` (disabling ASLR), `/NXCOMPAT:NO` (disabling DEP), `/guard:cf-` (disabling CFG) | Disabling exploit mitigations is suspicious unless justified |
| Hardcoded absolute paths to external tools/DLLs | Could reference attacker-controlled binaries |

### 2.3 — Pre-compiled binary audit
For every `.obj`, `.lib`, `.dll`, `.sys`, `.exe`, `.bin` file in the repository:
- **Is there corresponding source code?** If not → **HIGH** risk flag.
- **Does it match a known upstream build?** Compare hashes against official releases.
- **Run `strings`** on the binary to look for embedded URLs, IPs, command strings.

### 2.4 — Dependency integrity
For each folder under `/dependencies`, `/vendor`, `/third-party`, `/libraries`:
- Identify the upstream library (e.g., Zydis, Keystone, ia32-doc).
- Verify the version/commit matches a legitimate release.
- Check for modifications to the dependency source code.

**Output:** Build system section with per-file verdicts and a list of all pre-compiled binaries with their provenance status.

---

## Phase 3 — Static Source Code Analysis (Core Scan)

This is the main malware detection phase. Every source file (`.c`, `.cpp`, `.h`, `.hpp`, `.asm`, `.inl`, `.inc`) must be scanned.

### 3.1 — Automated pattern scan
Run the following regex-based detectors against all source files. For each match, record the file, line number, and context.

#### Category A: Process Injection APIs (Severity: CRITICAL)
```
CreateRemoteThread, NtCreateThreadEx, RtlCreateUserThread
VirtualAllocEx, NtAllocateVirtualMemory (with remote process handle)
WriteProcessMemory, NtWriteVirtualMemory
QueueUserAPC, NtQueueApcThread, NtQueueApcThreadEx
SetThreadContext, NtSetContextThread
```

#### Category B: Credential Harvesting & Keylogging (Severity: CRITICAL)
```
GetAsyncKeyState, SetWindowsHookEx, WH_KEYBOARD, WH_KEYBOARD_LL
LsaUnprotectMemory, CredEnumerate, CredRead
ReadProcessMemory targeting LSASS, SAM, SECURITY hive
MiniDumpWriteDump (targeting lsass.exe)
```

#### Category C: Rootkit / Kernel Hooks (Severity: HIGH)
```
SSDT, KeServiceDescriptorTable, KiServiceTable
ActiveProcessLinks, PsActiveProcessHead (DKOM)
KeInsertQueueApc (kernel APC injection)
ObRegisterCallbacks (process/thread protection)
CmRegisterCallback (registry hiding)
FltRegisterFilter (filesystem hiding)
```

#### Category D: Hypervisor-Specific Threats (Severity: HIGH — requires manual verification)
```
# MSR Abuse
IA32_LSTAR, IA32_SYSENTER_EIP — syscall handler redirection
IA32_EFER — long mode / NX bit manipulation
IA32_FEATURE_CONTROL — VMX lock manipulation
IA32_DEBUGCTL — debug control register manipulation
__readmsr, __writemsr, rdmsr, wrmsr — generic MSR access

# EPT / NPT Weaponization (Intel VT-x / AMD-V)
EptViolation, EptHook, HiddenHook — EPT hooking logic
Execute-only EPT entries — code invisible to memory scanners
Split-page EPT — different pages mapped for read vs execute
NPT (Nested Page Table) manipulation for AMD-V

# VMCALL / VMMCALL Backdoors
Custom hypercall handlers accepting magic numbers
Undocumented VMCALL dispatch cases — check the switch/case in the VM-exit handler
Any hypercall that accepts user-controlled pointers without validation

# Control Register Abuse
__readcr0, __writecr0 — disabling write protection (CR0.WP)
__readcr3, __writecr3 — address space switching
__readcr4, __writecr4 — enabling/disabling VMX, SMEP, SMAP

# VM-Exit Handler Analysis
The VM-exit handler (the main switch/case on exit reason) is the MOST CRITICAL function.
Every exit reason case must be reviewed:
  - CPUID exits: are they spoofing results?
  - MSR exits: are they redirecting system handlers?
  - EPT violation exits: are they implementing hidden hooks?
  - VMCALL exits: are they exposing a backdoor interface?
```

#### Category E: Interrupt & Exception Hijacking (Severity: HIGH)
```
IDT manipulation — custom handlers for #GP, #PF, #DB, #BP, #UD
NMI abuse — using Non-Maskable Interrupts for covert execution
IPI (Inter-Processor Interrupt) — forcing code execution on other cores
INT 2E / SYSCALL — raw system call invocation
```

#### Category F: Anti-Analysis / Evasion (Severity: MEDIUM)
```
RDTSC, RDTSCP — timing-based detection of analysis environments
CPUID hypervisor-present bit — checking/clearing to hide from OS
NtQuerySystemInformation(SystemKernelDebuggerInformation) — debugger detection
IsDebuggerPresent, CheckRemoteDebuggerPresent
ProcessDebugPort, ProcessDebugFlags, ProcessDebugObjectHandle
```

#### Category G: Networking & Exfiltration (Severity: MEDIUM)
```
WSAStartup, socket, connect, send, recv, bind, listen, accept
WinHTTP/WinINet: WinHttpOpen, InternetOpen, HttpSendRequest
URLDownloadToFile, URLDownloadToCacheFile
DNS: DnsQuery, getaddrinfo (for C2 resolution)
Hardcoded IPs (regex: \b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b)
Hardcoded domains or URLs (regex: https?://[^\s]+)
Base64 encoded strings (regex: [A-Za-z0-9+/]{20,}={0,2})
```

#### Category H: Privilege Escalation (Severity: MEDIUM)
```
SeDebugPrivilege, SE_DEBUG_NAME, SeTcbPrivilege
AdjustTokenPrivileges, OpenProcessToken, ImpersonateLoggedOnUser
NtSetInformationToken, DuplicateTokenEx
UAC bypass patterns: CMSTPLUA, fodhelper, eventvwr
```

#### Category I: Persistence Mechanisms (Severity: MEDIUM)
```
CreateService, OpenSCManager, StartService — driver/service installation
RegSetValue, RegCreateKey targeting Run/RunOnce keys
Scheduled tasks: schtasks, ITaskService, CLSID references
Boot-time: BcdEdit, bootmgr, winload modification
Registry callbacks for self-protection
```

#### Category J: Obfuscation & Encoded Payloads (Severity: HIGH)
```
Large byte arrays (>100 bytes) of hex values — potential shellcode
XOR loops against string data — runtime string decryption
constexpr string encryption — compile-time obfuscation
#include of .bin/.dat files — embedded binary data
High-entropy sections in source — compressed/encrypted payloads
RC4, AES, custom cipher implementations applied to strings/data (not for legitimate crypto)
```

### 3.2 — Manual deep review targets
After the automated scan, the following files/functions MUST be manually reviewed regardless of scan results:

1. **The VM-exit handler** — the main dispatch function (e.g., `HandleVmExit`, `SvHandleVmExit`, `VmmpHandleExit`). Review every case in the exit-reason switch.
2. **Driver entry point** (`DriverEntry`) — trace the full initialization path.
3. **Driver unload** (`DriverUnload`) — does it actually clean up, or is cleanup deliberately incomplete (persistence)?
4. **Any IOCTL dispatch** (`IRP_MJ_DEVICE_CONTROL` handler) — what operations can usermode request?
5. **Any function name containing**: `Hook`, `Hide`, `Inject`, `Spoof`, `Bypass`, `Steal`, `Dump`, `Exfil`, `Backdoor`, `Shell`, `Payload`, `Implant`.

**Output:** Per-category findings table with file, line, snippet, severity, and verdict (legitimate / suspicious / malicious).

---

## Phase 4 — Assembly Source Review

Assembly files (`.asm`, `.nasm`, `.s`) require special attention as they can contain logic invisible to C/C++ grep patterns.

### Checks:
| What to look for | Why |
|---|---|
| `syscall` / `sysenter` / `int 2Eh` instructions | Raw system call invocation, bypassing API hooking |
| `vmcall` / `vmmcall` instructions | Hypercall instructions — match against documented handlers |
| `cpuid` instruction with specific leaf values | Environment detection or result spoofing |
| `rdmsr` / `wrmsr` instructions | Direct MSR manipulation |
| `invlpg`, `invept`, `invvpid` | TLB/EPT invalidation — verify context |
| Shellcode patterns: `db 0x90, 0x90, ...` or long `db` sequences | Embedded machine code |
| `jmp` to hardcoded addresses | Could be jumping to injected code |
| Self-modifying code patterns | Code that rewrites itself at runtime |

**Output:** Assembly review section with annotated findings.

---

## Phase 5 — Resource & Data File Audit

### Checks:
- `.rc` (resource) files: do they embed executables, DLLs, or drivers?
- `.inf` files: do they install drivers or modify system configuration?
- `.reg` files: what registry keys do they modify?
- `.dat`, `.bin` files: run `file` command, check entropy, run `strings`.
- Image files: check for steganographic payloads (unusually large file sizes for the image dimensions).

**Output:** Data file inventory with provenance and risk assessment.

---

## Phase 6 — Dynamic Analysis (When Possible)

> ⚠️ **Only perform this phase in an isolated VM with no network access and a snapshot to revert to.**

### 6.1 — Pre-execution setup
- Snapshot the VM.
- Start Procmon with filters for the driver/process under test.
- Start Wireshark/tcpdump on all interfaces.
- Enable kernel debugging if possible (WinDbg + KD).

### 6.2 — Execution monitoring

| Tool | What to watch |
|---|---|
| **Procmon** | File creates/writes, registry modifications, process creation, thread creation |
| **Wireshark** | Any DNS queries, HTTP/HTTPS connections, raw TCP/UDP traffic |
| **WinDbg** | Breakpoints on key functions: `NtCreateFile`, `NtWriteFile`, `NtCreateKey`, `NtDeviceIoControlFile` |
| **API Monitor** | Hook `CreateService`, `RegSetValue`, `connect`, `send` |
| **PoolMon** | Unexpected kernel pool allocations |

### 6.3 — Post-execution checks
- Check for new files on disk (especially in `System32\drivers\`, `%TEMP%`, `%APPDATA%`).
- Check for new registry keys (especially `HKLM\SYSTEM\CurrentControlSet\Services\`, `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`).
- Check for new scheduled tasks.
- Check for new network connections.
- Revert to snapshot.

**Output:** Dynamic analysis section with timeline of observed behaviors.

---

## Phase 7 — Architecture Verification (Hypervisor-Specific)

Hypervisors **inherently** perform actions that look malicious. This phase determines whether each suspicious behavior is **legitimate or weaponized**.

### Legitimacy framework:

| Behavior | Legitimate if... | Malicious if... |
|---|---|---|
| MSR hooking (LSTAR) | Used for syscall tracing/debugging with configurable filters | Unconditionally redirects all syscalls with no user control |
| EPT hooking | Used for breakpoints, code coverage, function tracing | Creates hidden code pages invisible to OS memory scanners |
| CPUID spoofing | Hides hypervisor presence from anti-cheat for debugging | Hides hypervisor to avoid security product detection |
| CR3 switching | Used for process-specific memory introspection | Used to access credentials or sensitive data in other processes |
| Control register writes | Setting/clearing VMX enable, SMEP/SMAP for virtualization needs | Disabling write protection (CR0.WP=0) to patch kernel code |
| VMCALL interface | Documented commands for debugger operations | Undocumented backdoor commands for kernel R/W |
| Process notification callbacks | Tracking target process lifecycle for debugger attach/detach | Monitoring all process creation for data harvesting |
| KUSER_SHARED_DATA access | Spoofing timestamps for anti-debug evasion | No clear purpose; arbitrary kernel memory exposure |

### Key question for every suspicious construct:
> *"Does this behavior serve the stated purpose of the software, and is it controllable/configurable by the user?"*

If NO → flag as **SUSPICIOUS** or **MALICIOUS** depending on severity.

**Output:** Architecture verification table mapping each suspicious behavior to its justification.

---

## Phase 8 — Final Verdict

### Verdict checklist
Answer each question YES or NO:

| # | Question | Answer |
|---|---|---|
| 1 | Does the code contain process injection APIs (`CreateRemoteThread`, `VirtualAllocEx`, `WriteProcessMemory`) used against external processes? | |
| 2 | Does the code contain credential harvesting or keylogging functionality? | |
| 3 | Does the code make outbound network connections to non-documentation URLs? | |
| 4 | Does the code contain persistence mechanisms (services, run keys, scheduled tasks) beyond its own driver loading? | |
| 5 | Does the code contain functionality that deviates from the stated purpose of the project? | |
| 6 | Does the code contain undocumented VMCALL/backdoor interfaces? | |
| 7 | Does the code contain EPT/NPT weaponization (split-page attacks, execute-only hooks) beyond debugging use? | |
| 8 | Does the code download or execute external content during build or runtime? | |
| 9 | Does the code contain pre-compiled binaries without corresponding source? | |
| 10 | Does the code contain obfuscated payloads, shellcode arrays, or high-entropy embedded data? | |

### Verdict determination:
- **✅ SAFE** — All questions answered NO. Suspicious behaviors are explained by legitimate hypervisor/debugger functionality.
- **⚠️ SUSPICIOUS** — 1-2 questions answered YES but with possible legitimate justification. Requires extra caution.
- **❌ DANGEROUS** — Any question answered YES without legitimate justification. Do not compile or execute.

---

## Tools Used

| Tool | Purpose |
|---|---|
| `ripgrep` (rg) | Fast regex pattern scanning across all source files |
| `strings` | Extract readable strings from binary files |
| `diff` | Compare against upstream source code |
| `Ghidra` | Reverse engineer pre-compiled binaries (`.obj`, `.lib`, `.sys`) |
| `x64dbg` / `WinDbg` | Dynamic analysis and kernel debugging |
| `Wireshark` | Network traffic monitoring |
| `Procmon` | File system & registry activity logging |
| `7-Zip` | Archive extraction and hash verification |
| PowerShell / `_generate_report.ps1` | Automated static security scanning (included in repo) |
