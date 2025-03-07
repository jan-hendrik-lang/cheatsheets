# Windows Memory Forensics Cheat Sheet

## Tools for Acquiring Memory Dumps

- **winpmem**: Common tool for acquiring live memory. [Link GitHub](https://github.com/Velocidex/WinPmem)
- **DumpIt**: Lightweight memory acquisition tool for Windows. [Link Magnet Forensics](https://www.magnetforensics.com/de/resources/magnet-dumpit-fuer-windows/)
- **FTK Imager**: Can be used for forensic imaging and memory acquisition. [Link Exterro](https://www.exterro.com/digital-forensics-software/ftk-imager)
- **Magnet RAM Capture**: Free tool for acquiring RAM images. [Link Magnet](https://www.magnetforensics.com/resources/magnet-ram-capture/)
- **Virtual Machine**: Virtual Machines e.g. Hyper-V, VMWare, or Oracle VM Virtual Box often provides Memory Dumps.

## Tools for Memory Analysis

- **Strings**: Extracts human-readable strings from memory dumps for quick analysis. [Link SysInternals](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)
- **Volatility3**: A powerful framework for in-depth memory forensics and analysis. [Link GitHub](https://github.com/volatilityfoundation/volatility3)
- **MemProcFs**: A virtual filesystem for memory analysis, enabling structured access to processes, services, and registry. [Link GitHub](https://github.com/ufrisk/MemProcFS)
- **MemProcFS Analyzer**: A Tool for automated forensic analysis of windows memory dumps with MemProcFs. [Link GitHub](https://github.com/LETHAL-FORENSICS/MemProcFS-Analyzer)
- **Eric Zimmermann Tools**: Eric Zimmerman is a SANS Instructor and developed important tools to enhance digital forensics. [Link GitHub](https://ericzimmerman.github.io/#!index.md)
- **Analyse Registry Hives with RegRipper3.0**: Tool to extract and analyze key artifacts from the Windows Registry. [Link GitHub](https://github.com/keydet89/RegRipper3.0)
- **Bulk Extractor**: Tool used to scan and extract forensic artifacts from raw data sources. [Link GitHub](https://github.com/simsong/bulk_extractor)
- **NirSoft Tools**: Tools for system monitoring, forensics, and mutch more. [Link NirSoft](https://www.nirsoft.net/)

## String Analysis

- **Basic String Search:**
  ```bash
  strings example.dmp | grep -i malware\.exe
  strings example.dmp | grep -i 192\.168\.10\.*
  strings example.dmp | grep -i Admin
  ```
- **Extended Context (-A, -B, -C options)**:
  ```bash
  strings -tx example.dmp | grep -i malware.exe
  ```
  - `-A`: Shows context after the match.
  - `-B`: Shows context before the match.
  - `-C`: Shows context before and after the match.
- **Advanced Tools:**
  - **bstrings** (by Eric Zimmerman): More advanced and structured string search. [Link GitHub](https://github.com/EricZimmerman/bstrings)
  - **Sysinternals Strings**: Provides alternative string parsing options for Windows Users. [Link SysInternals](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)
  
## Windows Virtual Memory

- **Pagefile.sys**: Used for paging memory when RAM is full.
- **Swapfile.sys**: Used primarily by modern UWP applications.
- **Analysis Limitations:** Cannot be parsed with Volatility, but `strings` can extract data.
- **Useful for Extracting Passwords, IOCs (e.g., Mimikatz, Cobalt Strike).**

## Volatility 3 - Introduction

- **Uses Symbol-Based Analysis Instead of Profiles.**
- **Key Plugin:** `windows.info.Info` to extract metadata from an image.
- **Basic Command:**
  ```bash
  python3 vol.py -f FILENAME.vmem windows.info
  ```
- **List Commands**: These commands rely on structured OS-maintained lists to enumerate objects, such as processes or modules. They are faster and more reliable for detecting active objects but can be bypassed by malware that unlinks entries from these lists. E.g.:
  ```bash
  windows.pslist
  ```
- **Examples of Structured OS-Maintained Lists in Windows**:
  - PsActiveProcessHead (EPROCESS Active Process List) → Used by windows.pslist
    - A doubly linked list that tracks active processes in the system.
    - Each EPROCESS structure has a FLINK (forward link) and BLINK (backward link) pointer connecting it to other processes.
    - Malware can unlink a process from this list to hide it.
  - LoadedModuleList (PEB/Kernel Module List) → Used by windows.modules
    - Tracks loaded kernel drivers and modules.
    - Located in nt!PsLoadedModuleList for kernel modules or in the Process Environment Block (PEB) for user-mode modules.
    - Rootkits can unlink malicious drivers from this list to evade detection.
- **Scan Commands**: These commands use pool-scanning, meaning they search raw memory for object signatures or structures rather than relying on OS-managed lists. This technique helps detect hidden or unlinked objects, such as terminated or stealth processes. E.g.:
  ```bash
  windows.psscan
  ```

## Process Enumeration

- Detects hidden/unlinked processes.
- Finds previously terminated processes (potentially revealing persistence mechanisms).
- Hidden or unlinked processes may indicate **malware or rootkit activity.**

- **Active Processes:**
  ```bash
  windows.pslist
  ```
  - Follows doubly linked list.
  - Cannot detect hidden/unlinked processes (e.g., DKOM).
- **Process Tree Structure:**
  ```bash
  windows.pstree
  ```
  - Displays parent-child process relationships.
  - Cannot detect hidden/unlinked processes.
- **Hidden & Terminated Processes:**
  ```bash
  windows.psscan
  ```

## Enumerate Loaded DLLs
- **DLL List from Process Environment Block (PEB):**
  ```bash
  windows.dlllist --pid XXX
  ```
- **Differentiating Static vs. Dynamic DLLs:**
  - **Static:** Part of Import Address Table (IAT).
  - **Dynamic:** Injected later, often via malicious code.
- **Dump Suspicious DLLs:**
  ```bash
  windows.dlllist --pid XXX --dump
  ```
- **Useful for Identifying Malicious DLLs:**
  - Hash Analysis.
  - Reverse Engineering.
  - YARA Scans.

## Command Line and Handle Analysis

### Process Command Lines
- Extract command-line arguments used by a process.
  ```bash
  windows.cmdline --pid XXX
  ```
- **Suspicious Indicators:**
  - `powershell.exe -EncodedCommand <Base64>`.
  - `svchost.exe` missing `-k` flag.
  - Execution from unusual locations (e.g., `C:\Users\Admin\AppData\Local\Temp`).

### Process Handles
- Identify files, registry keys, and objects accessed by processes.
  ```bash
  windows.handles --pid XXX
  ```
- Useful for detecting **keyloggers**, malware accessing log files, etc.

## Security Token Analysis

- Determine which security tokens are assigned to a process.
  ```bash
  windows.getsids --pid XXX
  ```
- Important Security Identifiers (SIDs):
  - `S-1-5-18` System.
  - `S-1-5-19` Local Service.
  - `S-1-5-20` Network Service.
  - `S-1-5-domain-500` Administrator Account.
- Microsoft overview of well-known SIDs. [Link Microsoft](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids)
  
## Analyzing Network Activity:
- **Quick check for open Network Connections:**
  ```bash
  windows.netstat
  ```
- **More comprehensive check for open Network Connections:**
  ```bash
  windows.netscan
  ```
- Identifies suspicious connections to external servers (e.g., C2 communication).

## Registry Analysis

- **List Registry Hives:**
  ```bash
  windows.registry.hivelist
  ```
- **Identify Last Executed Programs:**
  ```bash
  windows.registry.printkey --key 'HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer'
  ```
- **Dump Registry Data:**
  ```bash
  windows.registry.hivelist --dump
  ```
- **Analyze Dumped Registry Data:**
  - Use RegRipper3.0
  ```bash
  .\rip.exe -r "path\to\dumped_registry_hive" -f PROFILE > output.txt
  ```
  - Use Eric Zimmerman´s Registry Explorer

## Code Injection Techniques

### Basic Code Injection:
- Malicious code is inserted into the address space of another process to execute under its privileges, often used by malware for stealth and persistence.
-  Identifies inconsistencies between different process memory structures (PEB, LDR lists, VAD) to detect stealthy code injections and hidden modules:
  ```bash
  windows.ldrmodules
  ```

### Reflective Code Injection:
- **Technique:** A malicious DLL is loaded into a process without using standard Windows API calls like `LoadLibrary`, allowing execution without being easily detected by conventional methods.
- Scan process memory for **Portable Executable (PE) Headers**.
  ```bash
  windows.malfind
  ```
- **Common Indicators of Injection:**
  - **MZ Headers in Unexpected Memory Regions.**
  - **Unusual Assembly Instructions (NOP, JMP, etc.).**

### Process Hollowing:
- **Technique:** A legitimate process is created in a suspended state, and its memory is unmapped and replaced with malicious code. Then resumed to execute under the guise of a trusted process.
- **Volatility Detection:**
  ```bash
  windows.hollowprocesses
  ```
  - Compare Process Environment Block (PEB) and Virtual Address Descriptor (VAD).


### System Service Descriptor Table (SSDT) Hook:
- **Technique:** Kernel-level technique where malware modifies the SSDT to redirect system calls (e.g., NtOpenProcess, NtCreateFile) to malicious code, often used for stealthy rootkits or privilege escalation.
- **Volatility Detection:**
  ```bash
  windows.ssdt
  ```

## Kernel Module and Rootkit Detection

### Enumerating Kernel Modules:
- **List Loaded Drivers:**
  ```bash
  windows.modules
  ```
- **Find Hidden/Unlinked Drivers:**
  ```bash
  windows.modscan
  ```
- **Detection of Malicious Drivers:**
  - Signed but compromised drivers used in ransomware attacks.
  - **Baseline Analysis:** Compare known good memory dumps to detect anomalies.

## Dumping

- **Use Cases:**
  - Extract credentials, encryption keys.
  - Identify malware artifacts.
  - Check Hashes.

- **Intersting Files:**
  - **Registry Hives:** `ntuser.dat`, `userclass.dat`, `SYSTEM.dat`, `SOFTWARE.dat`.
  - **Webbrowser History:** `edge`, `firefox`, `chrome` and analyse it with NirSoft Tools.
  - **Defender:** `defender` and analyse it with `net6` Tool from Eric Zimmerman.

### Dumping Processes
- **Extract a running process from memory:**
  ```bash
  windows.pslist --pid XXX --dump
  ```

### Dumping Files:
- **Locate Cached Files in Memory:**
  ```bash
  windows.filescan
  ```
- **Extract Files from Memory:**
  ```bash
  windows.dumpfiles --pid XXX
  ```

### Dumping Memory Sections:
- **Extract a Process's Memory:**
  ```bash
  windows.memmap --pid XXX --dump
  ```

### Dumping DLL's
- **Extract DLL´s:**
  ```bash
  windows.dlllist --dump
  ```

### Dumping Kernel Modules
- **Extract kernel modules (drivers):**
  ```bash
  windows.modules --dump
  ```

## YARA Rule-Based Detection

- **Scanning Memory with Custom YARA Rules:**
  ```bash
  windows.vadyarascan --yara-file custom_rule.yara
  ```
- **Install YARA Support:**
  ```bash
  pip install yara-python
  ```

## Advanced Tools

### MemProcFS
- Mounts memory images as a virtual filesystem.
- Provides structured access to processes, services, registry, etc.
- Automated forensic analysis of windows memory dumps with MemProcFS Analyzer

### WinDbg for Crash Dump Analysis
- **Load and analyze crash dumps:**
  ```
  .load <dmp_file>
  dx
  !process 0 0
  ```
