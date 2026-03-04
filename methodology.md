# 🔬 Analysis Methodology

Our community-driven malware analysis for hypervisor sources relies on a combination of rigorous static testing, dynamic behavior observation, and architectural validation.

## 1. Static Source Code Analysis
Every file in the repository is scanned and manually reviewed for suspicious logic.
- **Rootkit Indicators:** We look for System Service Descriptor Table (SSDT), Interrupt Descriptor Table (IDT), Global Descriptor Table (GDT) hooks, Hidden process/file manipulation, Direct Kernel Object Manipulation (DKOM), and manipulation of CR registers beyond hypervisor specifications.
- **Payload & Injection Indicators:** We review for hardcoded URLs, suspicious IP addresses, domains, Base64/XOR/RC4 encoded blobs, shellcode arrays, and WinAPI abuse (`CreateRemoteThread`, `VirtualAllocEx`, `WriteProcessMemory`).
- **Exfiltration & Surveillance:** We look for file system reads of `%APPDATA%`, `SAM`, `LSASS`, screenshot logic, and general outbound network routines untouched by legitimate features.
- **Privilege Escalation:** Analysis includes token impersonation manipulations, UAC bypass vectors, and `SeDebugPrivilege` / `SeTcbPrivilege` abuse.

## 2. Dependency and Supply Chain Audit
- **Build Scripts:** `CMakeLists.txt`, `Makefile`, `.ps1`, `.bat`, or `.sh` scripts are audited to ensure they do not fetch unverified external binaries or invoke malicious payloads during compilation.
- **Libraries:** Folders like `/dependencies` or `/vendor` are matched against upstream repositories to ensure they haven't been backdoored with poisoned libraries.

## 3. Dynamic Analysis & Execution Logging
Where possible, compiled instances of the hypervisor are executed within an isolated, heavily monitored sandbox to trace behavior that static analysis may not immediately reveal.
- **Network Traffic Monitoring:** Using Wireshark and proxy solutions to catalog all outbound DNS and HTTP(S) requests.
- **Filesystem & Registry Logging:** Using Procmon (Process Monitor) to trace the creation of persistence mechanisms (run keys, rogue services, bootloader modifications).

## 4. Architectural Verification
Hypervisors innately perform suspicious behaviors (e.g., trapping system calls, hooking EPTs, modifying MSRs). Part of the methodology involves verifying **why** these functions exist. If a hook serves a legitimate virtualization feature, it is documented as such. If it is extraneous, it is flagged as malware.
