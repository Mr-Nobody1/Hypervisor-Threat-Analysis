# Analysis Report: AFOP.B1-HypervisorSources

## Overview
**File:** `AFOP.B1-HypervisorSources.7z`
**Date Analyzed:** 2026-02-28
**Verdict:** ✅ SAFE
**Analyst:** Antigravity

## Summary of Analysis
Based on an exhaustive static code analysis of the provided `AFOP.B1-HypervisorSources` repository (specifically the `HypervisorIntelSource` and `HypervisorAMDSource` trees), this codebase is confirmed to be the source code for a hypervisor-based kernel debugger and a minimal AMD hypervisor respectively. Due to its nature, it inherently utilizes low-level subsystem manipulation, but it does **not** contain any malicious payloads, trojans, or data-stealing malware.

## Detailed Findings

### Intel Source (`HypervisorIntelSource`)
The Intel codebase matches the open-source **HyperDbg** project architecture.

1. **Build & Config Files (`CMakeLists.txt`, `.clang-format`, `_generate_report.ps1`)**
   - **Purpose:** Standard compilation and build rule configuration.
   - **Suspicious Findings:** The PowerShell script `_generate_report.ps1` scans for malware signatures, but there are no malicious build scripts pulling external unchecked binaries (no supply chain attack vectors).
   - **Risk Level:** CLEAN.

2. **Evading mechanisms (`hyperevade` e.g., `SyscallFootprints.c`)**
   - **Purpose:** Implements transparent debugging by avoiding detection from anti-cheat or anti-debugging mechanisms.
   - **Suspicious Findings:** `TransparentHandleNtOpenFileSyscall` and other functions purposefully corrupt CPU registers to trick the OS into hiding the debugger's presence.
   - **Risk Level:** CLEAN (Flagged as HIGH by static scanners due to rootkit-like evasion features, but legitimate for a transparent debugger).

3. **Hypervisor Core (`hyperhv` e.g., `MemoryManager.c`)**
   - **Purpose:** The core hypervisor engine handling Extended Page Tables (EPT) and Virtual Machine Control Structure (VMCS).
   - **Suspicious Findings:** Uses DKOM, `KeStackAttachProcess`, and physical memory mappings via `MmMapIoSpaceEx` and `Vmcall`s to read/write memory bypassing standard OS guarantees.
   - **Risk Level:** CLEAN (Legitimate VMX root-mode memory introspection logic).

4. **Kernel Debugger (`hyperkd` e.g., `Attaching.c`, `Thread.c`, `UserAccess.c`)**
   - **Purpose:** The kernel-debugger module responsible for halting the OS and enumerating active processes/threads.
   - **Suspicious Findings:** Employs hidden thread execution interception, unexported NTDLL/ntoskrnl function queries (`PsGetProcessPeb`, `ZwQueryInformationProcess`), and CR register swaps.
   - **Risk Level:** CLEAN.

5. **User-mode Engines (`hyperdbg`, `libhyperdbg`, `script-eval`)**
   - **Purpose:** The UI and command parsing engines, evaluating user commands (like `$peb`, `$pname`).
   - **Suspicious Findings:** `PseudoRegisters.c` uses dynamic address resolution (`GetProcAddress` on `NtQueryInformationProcess`) to fetch the PEB manually.
   - **Risk Level:** CLEAN.

6. **Dependencies & Symbol Parsing (`dependencies`, `symbol-parser`)**
   - **Purpose:** Third-party compilation tools (Zydis for disassembly) and PDB symbol downloading.
   - **Suspicious Findings:** `URLDownloadToFileA` is used inside `symbol-parser.cpp`.
   - **Risk Level:** CLEAN (It exclusively contacts official symbol servers via HTTP to fetch `.pdb` files necessary for kernel debugging; it does not download malicious shellcode).

### AMD Source (`HypervisorAMDSource`)
The AMD module is located entirely in the `SimpleSvm` folder and consists of only three core source files (`SimpleSvm.cpp`, `SimpleSvm.hpp`, and `x64.asm`).

- **Purpose:** It is a minimal implementation of an AMD-V (Secure Virtual Machine / SVM) hypervisor. The code heavily references known open-source academic hypervisors such as *SimpleVisor* (by Alex Ionescu) and *HelloAmdHvPkg* (by tandasat).
- **Rootkit / Evasion:** It contains standard AMD-V initialization routines (`VMRUN`, `VMCB` setups) and intercepts standard system events (like CPUID), but does not contain extensive stealth modifications.
- **Malicious Payload Indicators:** ZERO. There are no mentions of thread injection logic such as `CreateRemoteThread` or `WriteProcessMemory`.
- **Data Exfiltration / Networking:** ZERO. The only URLs found in the codebase are links to Microsoft MSDN documentation, AMD architecture manuals, and GitHub repositories for reference.
- **Persistence:** ZERO.

## Security Indicators Reviewed
*   **Malicious Payload Indicators:** ZERO. `CreateRemoteThread`, `VirtualAllocEx`, and `WriteProcessMemory` do not exist anywhere in the codebase for process injection.
*   **Data Exfiltration:** ZERO logic exists for scraping `%APPDATA%`, passwords, or broadcasting system information outbound. No reverse shells or suspicious IPs mapped. 
*   **Persistence Mechanisms:** ZERO. Does not attempt to install latent bootkits, hidden run keys, or rogue services intended to survive secretly across reboots.

## Conclusion
The codebase is clean. The low-level logic, hook placements, and subsystem manipulation are consistent with the requirements of a virtualization-based debugger.
