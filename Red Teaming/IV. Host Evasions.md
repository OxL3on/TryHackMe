# Windows Internals

### --------------------------------- Processes -----------------------------------

## **Processes (Windows)**

**Process:** A running program instance. One application can create multiple processes.

Examples: `MsMpEng` (Defender), `wininit` (Windows initialization), `lsass` (authentication).

**Process attacks:** Injection (`T1055`), Hollowing (`T1055.012`), Masquerading (`T1055.013`).

### **Process Components**

| Component                     | Purpose                  |
| ----------------------------- | ------------------------ |
| Private Virtual Address Space | Process memory           |
| Executable Program            | Code and data            |
| Open Handles                  | System resources         |
| Security Context              | User, groups, privileges |
| PID                           | Unique process ID        |
| Threads                       | CPU execution units      |

### **Process Memory**

| Component         | Purpose                 |
| ----------------- | ----------------------- |
| Code              | Instructions            |
| Global Variables  | Stored variables        |
| Process Heap      | Dynamic data            |
| Process Resources | Additional resources    |
| Environment Block | Environment information |

### **Task Manager Information**

| Value     | Example       |
| --------- | ------------- |
| Name      | `conhost.exe` |
| PID       | `7408`        |
| Status    | `Running`     |
| User Name | `SYSTEM`      |

### **Tools**

**Process Hacker 2**, Process Explorer (process inspection), Procmon (system activity monitoring).

**Remember:** Processes have memory, threads, handles and security context, making them targets for injection, masquerading and privilege abuse.


### --------------------------------- Threads -----------------------------------

## **Threads (Windows)**

**Thread:** An execution unit of a process, scheduled by the CPU. It controls the execution of a process based on CPU, memory, priority, and other factors.

Threads are common targets for code execution and can be abused alone or combined with other API calls. Threads share their parent process's code, global variables, and resources.

**Thread components:** Stack (calls, exceptions, thread data), Thread Local Storage (thread-specific storage pointers), Stack Argument (unique thread values), Context Structure (CPU register values maintained by kernel).


### --------------------------------- Virtual Memory -----------------------------------

## **Virtual Memory (Windows)**

**Virtual Memory:** Each process gets a private virtual address space. The Memory Manager translates virtual addresses → physical addresses, preventing memory collisions between processes. When RAM is insufficient, memory pages are moved to disk (paging).

**Address space:**

- **32-bit:** 4 GB total, User space `0x00000000–0x7FFFFFFF` (2 GB), System space `0x80000000–0xFFFFFFFF` (2 GB). Can increase User space up to 3 GB using `increaseUserVA` / `AWE` (Address Windowing Extensions).
- **64-bit:** Theoretical 256 TB (according to the source), much larger memory space, reducing address-space limitations.

**Remember:** Virtual memory provides isolation, address translation and paging, allowing processes to safely use memory without directly accessing physical RAM.


### --------------------------------- Dynamic Link Libraries -----------------------------------

## **Dynamic Link Libraries (DLL)**

**DLL:** Shared library containing code/data used by multiple programs, improves modularity, code reuse, memory efficiency and reduces disk usage. Programs depend on DLLs for functionality, making them potential attack targets.

**DLL attacks:** Hijacking (`T1574.001`), Side-Loading (`T1574.002`), Injection (`T1055.001`).

**DLL linking:**

- **Load-time linking:** DLL functions called directly, requires `.h` header and `.lib` import library.
- **Run-time linking:** Load DLL using `LoadLibrary()` / `LoadLibraryEx()`, locate function using `GetProcAddress()`, unload using `FreeLibrary()`.

**Key functions:** `DllMain()` (DLL entry point), `HelloWorld()` (example exported function), `LoadLibrary()`, `GetProcAddress()`, `FreeLibrary()`.

**Remember:** Run-time dynamic linking allows a program to load DLLs when needed, offering flexibility and making it useful in malicious code.

**DLL example:**

```
BOOL APIENTRY DllMain(HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    return TRUE;
}

void HelloWorld()
{
    MessageBox(NULL, TEXT("Hello World"), TEXT("In a DLL"), MB_OK);
}
```

**Run-time loading:** `LoadLibrary()` → `GetProcAddress()` → call function → `FreeLibrary()`.


### --------------------------------- Portable Executable Format -----------------------------------

## **Portable Executable (PE) Format**

**PE:** Windows executable/object file structure that defines how code, data, and other information are stored. Includes PE and COFF formats.

<img width="441" height="521" alt="image" src="https://github.com/user-attachments/assets/4ad5cd96-e747-4a47-9b61-5edc6bc9af41" />

**PE structure (7 components):**

1. **DOS Header** – Identifies file format, starts with `MZ`.
2. **DOS Stub** – Displays `This program cannot be run in DOS mode`.
3. **PE File Header** – PE signature (`PE`) and file information.
4. **Image Optional Header** – Important loading information, includes Data Directories.
5. **Data Dictionaries** – Point to image data directories.
6. **Section Table** – Defines file sections and their properties.
7. **Sections** – Contains actual code and data.

**Sections:**

| Section             | Purpose                                 |
| ------------------- | --------------------------------------- |
| `.text`             | Executable code, entry point            |
| `.data`             | Initialized variables/data              |
| `.rdata` / `.idata` | Read-only data, imports/DLL information |
| `.reloc`            | Relocation information                  |
| `.rsrc`             | Resources (images, etc.)                |
| `.debug`            | Debug information                       |

**Remember:** PE headers define the file's structure and loading information; sections contain its actual contents.


### --------------------------------- Interacting with Windows Internals -----------------------------------

## **Interacting with Windows Internals**

**Windows API:** Provides applications access to Windows OS functionality (Win32 API), allowing interaction with OS components, memory and hardware.

**Kernel:** Controls processes and bridges software ↔ hardware interactions.

### **Processor Modes**

| User Mode                     | Kernel Mode                  |
| ----------------------------- | ---------------------------- |
| No direct hardware access     | Direct hardware access       |
| Private virtual address space | Shared virtual address space |
| Access to owned memory        | Access to physical memory    |

<img width="849" height="866" alt="image" src="https://github.com/user-attachments/assets/ba46f000-4bbc-4d3a-92f4-24467ae41b26" />

**Switching Point:** System/API calls switch execution from User Mode → Kernel Mode. Languages may pass through a runtime first (e.g., C# → CLR → Win32 API → System Call).

### **Memory Interaction**

Basic flow: Allocate memory → Write data → Execute from memory.

**APIs:** `OpenProcess` (get process handle), `VirtualAllocEx` (allocate memory), `WriteProcessMemory` (write memory), `CreateRemoteThread` (create execution thread).

**THM:** [Windows API Room](https://tryhackme.com/room/windowsapi?utm_source=chatgpt.com)




# Introduction to Windows API

### --------------------------------- Subsystem and Hardware Interaction -----------------------------------

### **Subsystem & Hardware Interaction**

**Win32 API:** Interface between user-mode applications and Windows kernel, allowing controlled access to OS subsystems and hardware.

<img width="849" height="866" alt="image" src="https://github.com/user-attachments/assets/1262c741-36b7-4271-b86f-940738d11fbc" />

**Processor modes:**

- **User mode:** No direct hardware access, access only to permitted/owned memory.
- **Kernel mode:** Direct hardware access, access to physical memory.

**Switching point:** API/System calls allow requests to transition from User Mode → Kernel Mode for processing.

**Flow:** Application → Runtime (if applicable) → Win32 API → System Call → Kernel.

**Resources:** [Windows Internals](https://tryhackme.com/room/windowsinternals?utm_source=chatgpt.com), [Runtime Detection Evasion](https://tryhackme.com/room/runtimedetectionevasion?utm_source=chatgpt.com).


### --------------------------------- Components of the Windows API -----------------------------------

### **Components of the Windows API**

**Windows API (Win32)** is organized into several layers:

**Structure (top → bottom):** API → Header files/Imports → Core DLLs → Supplemental DLLs → Call Structures → API Calls → Input/Output Parameters.

- **API:** General interface to Windows functionality.
- **Header files/Imports:** Define libraries and function declarations, used to access function addresses.
- **Core DLLs:** `KERNEL32`, `USER32`, `ADVAPI32` — provide core OS services.
- **Supplemental DLLs:** Additional functionality, e.g. `NTDLL`, `COM`, `FVEAPI` (\~36 DLLs).
- **Call Structures:** Define API functions and required parameters.
- **API Calls:** Functions called by programs.
- **In/Out Parameters:** Values passed into or returned from API calls.


### --------------------------------- OS Libraries -----------------------------------

### **OS Libraries**

Win32 API functions reside in memory and are accessed through **function pointers**. ASLR randomizes memory addresses, so programs need a way to locate API functions at runtime.

**Two common methods:** Windows Header (`windows.h`), P/Invoke.

**Windows Header (**`windows.h`**):** Loader resolves function addresses at runtime using a thunk table, allowing unmanaged programs to call Win32 functions by including `windows.h`.

**P/Invoke (Platform Invoke):** Allows managed code (e.g., C#) to call unmanaged DLL functions. Uses `DllImport` to import DLLs and `extern` to declare external methods.

```
[DllImport("user32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
```

```
private static extern int MessageBox(
    IntPtr hWnd, string lpText, string lpCaption, uint uType);
```

**Remember:** `DllImport` imports the DLL, `extern` declares its function, and calling the managed method invokes the unmanaged Win32 function.


### --------------------------------- API Call Structure -----------------------------------

### **API Call Structure**

**API calls** provide Windows functionality and are defined by function names and their input/output parameters.

**Naming conventions:** `A` = ANSI (8-bit), `W` = Unicode, `Ex` = extended version with additional functionality/parameters.

**I/O parameters:** `[in]` = input, `[out]` = output, defined by each API's call structure.

Example: `WriteProcessMemory()` — writes data to a process's memory.

```
BOOL WriteProcessMemory(
  [in]  HANDLE  hProcess,
  [in]  LPVOID  lpBaseAddress,
  [in]  LPCVOID lpBuffer,
  [in]  SIZE_T  nSize,
  [out] SIZE_T  *lpNumberOfBytesWritten
);
```

**Parameters:** `hProcess` (target process handle), `lpBaseAddress` (destination address), `lpBuffer` (data source), `nSize` (bytes to write), `lpNumberOfBytesWritten` (bytes successfully written).

**Remember:** Always check API documentation and examples to understand parameters, accepted values and function behavior.

**References:** [Windows API Documentation](https://docs.microsoft.com/en-us/windows/win32/apiindex/windows-api-list?utm_source=chatgpt.com), [PInvoke](http://pinvoke.net/?utm_source=chatgpt.com), [WriteProcessMemory](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory?utm_source=chatgpt.com).


### --------------------------------- C API Implementations -----------------------------------

### **C API Implementations**

**C/C++** can directly access Win32 API calls using the Windows header:

```
#include <windows.h>
```

**Example API:** `CreateWindowExA()` creates a Windows GUI window.

**Important parameters:** `dwExStyle` (extended style), `lpClassName` (window class), `lpWindowName` (title), `dwStyle` (window style), `X/Y` (position), `nWidth/nHeight` (size), `hWndParent` (parent), `hMenu` (menu), `hInstance` (instance), `lpParam` (additional data).

**Example:** `CreateWindowEx(..., "Hello THM!", ...)` → creates a window titled **Hello THM!**.

**Remember:** C/C++ can interact with Windows APIs directly through `windows.h`; it's widely used because it provides low-level access and flexibility.


### --------------------------------- .NET and PowerShell API Implementations -----------------------------------

### **.NET & PowerShell API Implementations**

**P/Invoke:** Allows managed languages (.NET/C#) to call unmanaged Win32 API functions using `DllImport`, function declarations (`extern`) and parameter structures.

**.NET:** Import DLL → declare API function → call it like a normal method.

```
[DllImport("kernel32")]
public static extern IntPtr GetComputerNameA(StringBuilder lpBuffer, ref uint lpnSize);
```

Example: `GetComputerNameA()` retrieves the computer's name.

**PowerShell:** Uses `Add-Type` to compile and load Win32 API definitions into a .NET type.

```
$Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -NameSpace 'Win32' -PassThru
```

**API calls defined:** `GetProcAddress` (function address), `GetModuleHandle` (module handle), `VirtualProtect` (memory protection).

Call syntax:

```
[Win32.Kernel32]::<Imported Call>()
```

**Remember:** C# uses `DllImport` + `extern`; PowerShell uses API definitions + `Add-Type` to access Win32 functions.


### --------------------------------- Commonly Abused API Calls -----------------------------------

### **Commonly Abused API Calls**

Malware often abuses legitimate Windows APIs for malicious purposes. Resources for researching API abuse: [SANS](https://www.sans.org/white-papers/33649/?utm_source=chatgpt.com), [MalAPI.io](http://malapi.io/?utm_source=chatgpt.com).

| API                  | Purpose                              |
| -------------------- | ------------------------------------ |
| `LoadLibraryA`       | Load a DLL into the calling process  |
| `GetUserNameA`       | Get current thread's username        |
| `GetComputerNameA`   | Get local computer name              |
| `GetVersionExA`      | Get Windows version                  |
| `GetModuleFileNameA` | Get module/process file path         |
| `GetStartupInfoA`    | Get process startup information      |
| `GetModuleHandle`    | Get handle of loaded module          |
| `GetProcAddress`     | Get address of exported DLL function |
| `VirtualProtect`     | Change memory protection             |

**Remember:** These APIs are legitimate Windows functions, but malware can abuse them for DLL loading, information gathering, locating functions, and modifying memory permissions.


### --------------------------------- Malware Case Study -----------------------------------

## **Malware Case Study**

### **1. Keylogger**

**Goal:** Monitor keyboard events using a Windows hook.

| API                   | Purpose                           |
| --------------------- | --------------------------------- |
| `SetWindowsHookEx`    | Install a hook to monitor events  |
| `UnhookWindowsHookEx` | Remove the installed hook         |
| `GetModuleHandle`     | Get module handle                 |
| `GetCurrentProcess`   | Get current process pseudo-handle |

- `WHKEYBOARDLL = 13` → Low-level keyboard hook
- `SetHook()` installs the hook, `Application.Run()` keeps the application running, `UnhookWindowsHookEx()` removes it before exit.

### **2. Shellcode Launcher**

**Goal:** Allocate memory, place shellcode into memory, then execute it in a new thread.

| API                   | Purpose                             |
| --------------------- | ----------------------------------- |
| `VirtualAlloc`        | Allocate virtual memory             |
| `WaitForSingleObject` | Wait for an object/thread to finish |
| `CreateThread`        | Create a new thread                 |

**Execution flow:** `VirtualAlloc` → `Marshal.Copy` → `CreateThread` → `WaitForSingleObject`

- `MEM_COMMIT = 0x1000` → Commit memory
- `PAGE_EXECUTE_READWRITE = 0x40` → Memory is readable, writable and executable (RWX)
- `Marshal.Copy` → Copy shellcode into allocated memory

**Remember:** Keylogger uses hooks to monitor keyboard events, shellcode launcher uses memory allocation and thread creation to execute code.




# Abusing Windows Internals

### --------------------------------- Abusing Processes -----------------------------------

### **Process Components**

A process contains **Private Virtual Address Space** (allocated memory), Executable (code/data), Open Handles (system resources), Security Context (user/groups/privileges), PID (unique ID), Threads (execution units).

### **Process Injection Types**

| Type                                                                                                | Meaning                                        |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| [Process Hollowing](https://attack.mitre.org/techniques/T1055/012/?utm_source=chatgpt.com)          | Inject code into a suspended, hollowed process |
| [Thread Execution Hijacking](https://attack.mitre.org/techniques/T1055/003/?utm_source=chatgpt.com) | Hijack a suspended thread                      |
| [DLL Injection](https://attack.mitre.org/techniques/T1055/001/?utm_source=chatgpt.com)              | Inject a DLL into process memory               |
| [PE Injection](https://attack.mitre.org/techniques/T1055/002/?utm_source=chatgpt.com)               | Inject a PE image into a target process        |

All types: [MITRE T1055 – Process Injection](https://attack.mitre.org/techniques/T1055/?utm_source=chatgpt.com)

### **Shellcode Injection (4 Steps)**

1. `OpenProcess` → Open target process and obtain handle.
2. `VirtualAllocEx` → Allocate memory inside target process.
3. `WriteProcessMemory` → Write shellcode into allocated memory.
4. `CreateRemoteThread` → Execute code through a new thread.

<img width="841" height="301" alt="image" src="https://github.com/user-attachments/assets/6e13ab1c-c6af-4193-8d74-eb414e323b6d" />

### **Important APIs**

| API                  | Purpose                                   |
| -------------------- | ----------------------------------------- |
| `OpenProcess`        | Open target process, obtain handle/access |
| `VirtualAllocEx`     | Allocate memory in another process        |
| `WriteProcessMemory` | Write data into another process's memory  |
| `CreateRemoteThread` | Create thread inside target process       |

**Remember:** Process injection abuses legitimate Windows functionality to execute code inside another process, often manipulating its memory and threads.


### --------------------------------- Expanding Process Abuse -----------------------------------

### **Process Hollowing**

**Process Hollowing:** A technique where a legitimate suspended process is hollowed out (its original memory image is unmapped) and replaced with a malicious PE image.

### **6 Steps**

1. `CreateProcessA` → Create a process in suspended state.
2. `CreateFileA` → Open the malicious PE file.
3. `ZwUnmapViewOfSection` → Unmap the original process image.
4. `VirtualAllocEx` + `WriteProcessMemory` → Allocate memory and write PE headers/sections.
5. `SetThreadContext` → Set the thread's execution entry point.
6. `ResumeThread` → Resume the suspended process.

<img width="941" height="411" alt="image" src="https://github.com/user-attachments/assets/bfc099ae-0c64-420b-92ee-398b8e43d058" />

### **Important APIs**

| API                    | Purpose                             |
| ---------------------- | ----------------------------------- |
| `CreateProcessA`       | Create a suspended process          |
| `CreateFileA`          | Open the malicious file             |
| `GetFileSize`          | Get file size                       |
| `VirtualAlloc`         | Allocate local memory               |
| `ReadFile`             | Read malicious PE into local memory |
| `GetThreadContext`     | Retrieve thread registers/context   |
| `ReadProcessMemory`    | Read target process memory          |
| `ZwUnmapViewOfSection` | Unmap original process image        |
| `VirtualAllocEx`       | Allocate memory in target process   |
| `WriteProcessMemory`   | Write PE headers and sections       |
| `SetThreadContext`     | Change thread execution context     |
| `ResumeThread`         | Resume suspended thread             |

### **PE Concepts**

- `e_lfanew` → Offset from DOS header to PE header.
- `SizeOfImage` → Total image size in memory.
- `SizeOfHeaders` → Size of PE headers.
- `NumberOfSections` → Number of PE sections.
- `AddressOfEntryPoint` → Entry point RVA.
- `VirtualAddress` → Section's address in memory.
- `PointerToRawData` → Section's offset in the file.

**Remember:** Shellcode injection inserts shellcode into a process, while process hollowing replaces the original process image with another PE image.


### --------------------------------- Abusing Process Components -----------------------------------

## **Thread Execution Hijacking**

**Thread hijacking:** Instead of creating a new thread, malware hijacks an existing thread and changes its execution flow.

### **Process (Injection) Steps**

1. `OpenProcess` → Open target process.
2. `VirtualAllocEx` → Allocate memory in target.
3. `WriteProcessMemory` → Write code into allocated memory.
4. `CreateToolhelp32Snapshot` → Take a snapshot of system threads.
5. `Thread32First` / `Thread32Next` → Enumerate threads and find the target PID.
6. `OpenThread` → Open the target thread.
7. `SuspendThread` → Suspend the thread.
8. `GetThreadContext` → Retrieve thread registers/context.
9. Modify `RIP` → Redirect instruction pointer to another memory location.
10. `SetThreadContext` → Apply modified context.
11. `ResumeThread` → Resume execution.

### **Important APIs**

| API                        | Purpose                                  |
| -------------------------- | ---------------------------------------- |
| `CreateToolhelp32Snapshot` | Create a snapshot for thread enumeration |
| `Thread32First`            | Get the first thread in the snapshot     |
| `Thread32Next`             | Get subsequent threads                   |
| `OpenThread`               | Open a thread                            |
| `SuspendThread`            | Suspend execution                        |
| `GetThreadContext`         | Retrieve CPU/thread context              |
| `SetThreadContext`         | Update thread context                    |
| `ResumeThread`             | Resume thread execution                  |

### **Key Concept**

- `RIP` = Instruction Pointer on x64, determines the next instruction to execute.
- Thread hijacking redirects an existing thread's execution instead of creating a new one.
- Unlike `CreateRemoteThread`, hijacking reuses an existing thread.

**Remember:** Shellcode injection creates a new thread, while thread hijacking manipulates an existing thread's context and execution flow.


### --------------------------------- Abusing DLLs  -----------------------------------

## **DLL Injection**

**DLL Injection:** Injecting and loading a malicious DLL inside another process.

### **Process**

1. `CreateToolhelp32Snapshot` → Snapshot running processes.
2. `Process32First` / `Process32Next` → Enumerate processes and find target PID.
3. `OpenProcess` → Open the target process.
4. `VirtualAllocEx` → Allocate memory for the DLL path.
5. `WriteProcessMemory` → Write the DLL path into target memory.
6. `GetModuleHandle` + `GetProcAddress` → Locate `LoadLibraryA`.
7. `CreateRemoteThread` → Execute `LoadLibraryA` in the target process to load the DLL.

### **Important APIs**

| API                        | Purpose                           |
| -------------------------- | --------------------------------- |
| `CreateToolhelp32Snapshot` | Create process snapshot           |
| `Process32First`           | Get first process entry           |
| `Process32Next`            | Get next process entry            |
| `OpenProcess`              | Open target process               |
| `VirtualAllocEx`           | Allocate memory inside target     |
| `WriteProcessMemory`       | Write DLL path into target memory |
| `GetModuleHandle`          | Get module handle                 |
| `GetProcAddress`           | Find function address             |
| `LoadLibraryA`             | Load DLL                          |
| `CreateRemoteThread`       | Create a thread in target process |

**Remember:** DLL injection loads a DLL into another process, commonly by placing its path in target memory and triggering the DLL loader through `LoadLibraryA`.


### --------------------------------- Memory Execution Alternatives -----------------------------------

## **Memory Execution Alternatives**

Alternative methods of executing code besides `CreateThread` / `CreateRemoteThread`.

### **1. Function Pointer**

`((void(*)())addressPointer)();`

- Casts a memory address into a function pointer, then invokes it.
- Uses no additional API call for execution.
- Works with locally allocated memory.

### **2. Asynchronous Procedure Call (APC)**

- **APC** = function executed asynchronously in the context of a particular thread.
- `QueueUserAPC` queues an APC to a thread.
- APC executes when the thread enters an **alertable state**.
- Alertable state can occur while waiting for callbacks, e.g., `WaitForSingleObject` or `Sleep`.

Related APIs: `QueueUserAPC`, `ResumeThread`, `WaitForSingleObject`.

### **3. Section Manipulation**

Manipulates PE structures and sections to work with executable data.

- PE sections: `.text` (code), `.data` (data)
- PE dump can be examined using `xxd`.
- Uses offsets and calculations to navigate raw PE data.

Common techniques: **RVA Entry Point Parsing**, **Section Mapping**, **Relocation Table Parsing**.

**Remember:** Execution can use function pointers, APCs, or PE section manipulation instead of traditional thread creation.


### --------------------------------- Case Study in Browser Injection and Hooking -----------------------------------

## **Case Study: Browser Injection & Hooking (TrickBot)**

**TrickBot** is banking malware that injects into browser processes and hooks API functions to intercept sensitive information, such as credentials.

### **Injection Flow**

1. `OpenProcess` → Open target browser process.
2. `VirtualAllocEx` → Allocate memory inside the process.
3. `WriteProcessMemory` → Copy hooking function into memory.
4. `WriteProcessMemory` → Copy shellcode into memory.
5. `FlushInstructionCache` → Flush instruction cache.
6. `RemoteThread` → Create remote thread.
7. `ResumeThread` / `RtlCreateUserThread` → Resume or create thread.

### **Target Browsers**

`chrome.exe`, `iexplore.exe`, `firefox.exe`, `microsoftedgecp.exe`

### **Hook Installation**

- **Function pointers** → Identify function addresses and calculate offsets.
- `VirtualProtectEx` → Change memory protection to `PAGE_EXECUTE_READWRITE (0x40)` so code can be modified.
- `0xE9` → x86 relative JMP opcode.
- Hook rewrites the original function to redirect execution to the malware's hook function.
- `VirtualProtectEx` → Restore original memory protection after modification.

### **Main Takeaway**

TrickBot uses **reflective injection + API hooking** to inject itself into browser processes and redirect function execution, allowing it to intercept sensitive browser activity.

Injection puts code inside a process; hooking redirects a function's execution to different code.




# Introduction to Antivirus

### --------------------------------- Antivirus Software -----------------------------------

## **Antivirus Software (AV)**

**AV:** Host-based security software that runs in real-time, scans files, and detects/prevents malicious files from executing or spreading.

**What AV detects:** Malware that steals passwords, gains unauthorized access, encrypts/damages files, installs other malware/adware, or uses compromised systems for attacks (e.g., botnets).

**AV vs EDR:** AV scans, detects, and removes malicious files, EDR uses behavioral analytics to monitor file activity, memory, network connections, registry, and processes.

**History:** McAfee VirusScan (1987) was an early antivirus, designed to remove the Brain virus. Early AVs were CLI scanners; modern AVs use GUI and support Windows, Linux, macOS, Android, and iOS, with features like Anti-Exploit, Firewall, and Encryption.

- **AV Engine:** Core component responsible for analyzing and removing malicious files.
- **Scanner:** Scans files/directories in real-time or on-demand.
- **Detection techniques:** Signature, heuristic, and dynamic detection.
- **Compressors & Archives:** Scans compressed files (ZIP, TGZ, 7z, XAR, RAR, etc.).
- **PE Parsing & Unpackers:** Parses PE files and unpacks packed malware (UPX, Armadillo, ASPack, etc.) for analysis.
- **Emulator:** Executes suspicious files in a controlled, isolated environment and monitors API calls, memory, file changes, logs, processes, and web requests.

**Other AV features:** Self-protection driver, firewall/network inspection, CLI/GUI, daemon/service, management console.


### --------------------------------- Antivirus Features -----------------------------------

## **Antivirus Features**

**AV Engine:** Core component that detects and removes malware. Main features:

- **Scanner:** Real-time or on-demand scanning of files/directories, memory, registry, emails, vulnerabilities (GUI/CLI).
- **Detection techniques:** Signature-based (known patterns), Heuristic (suspicious behavior/characteristics), Dynamic (monitoring APIs/system calls in an isolated environment).
- **Compressors & Archives:** Scans compressed files such as ZIP, TGZ, 7z, XAR, RAR by extracting and inspecting contents.
- **PE Parsing & Unpackers:** Unpacks packed/encrypted executables (UPX, Armadillo, ASPack), analyzes PE headers to identify executable structure and resources.
- **Emulator:** Runs suspicious files in an isolated environment and collects API calls, memory dumps, filesystem changes, logs, processes, web requests, then stops when enough artifacts are collected.

**Other features:** Self-protection driver, Firewall/network inspection, CLI/GUI, Daemon/service, Management console.


### --------------------------------- AV Static Detection -----------------------------------

## **AV Static Detection**

**Static detection:** Detects malware by matching predefined signatures, strings, byte patterns, CRCs, or hashes (MD5/SHA1). Works best against known malware; signature databases must be updated.

<img width="800" height="470" alt="image" src="https://github.com/user-attachments/assets/d676ebd3-9632-43ff-b472-95abc8b78b36" />

### **ClamAV Results**

`clamscan.exe` scans files using its signature database.

| File            | Result                               |
| --------------- | ------------------------------------ |
| `AV-Check.exe`  | OK                                   |
| `backdoor1.exe` | `Win.Malware.Swrort-9872015-0 FOUND` |
| `backdoor2.exe` | OK (missed malware)                  |
| `eicar.com`     | `Win.Test.EICAR_HDB-1 FOUND`         |
| `notes.txt`     | OK                                   |

`backdoor1` detected because its shellcode pattern was known, `backdoor2` missed because XOR encryption changed the shellcode's byte patterns.

### **MD5 Signatures**

MD5 signature format: `Hash:Size-in-byte:FileName`

```
sigtool.exe --md5 eicar.com
```

EICAR MD5: `44d88612fea8a8f36de82e1278abb02f`

```
sigtool.exe --md5 backdoor2.exe
```

Output:\
`75047189991b1d119fdb477fef333ceb:6144:backdoor2.exe`

### **Custom ClamAV Database (.hdb)**

Create a custom signature database:

```
sigtool.exe --md5 backdoor2.exe > thm.hdb
```

Scan using the custom database:

```
clamscan.exe -d thm.hdb backdoor2.exe
```

Result: `backdoor2.exe.UNOFFICIAL FOUND`

### **YARA Static Detection**

**YARA:** Rule-based detection using unique strings, byte patterns, etc.

Analyze strings in a binary:

```
strings AV-Check.exe | findstr pdb
```

Example unique string:

`C:\Users\thm\source\repos\AV-Check\AV-Check\obj\Debug\AV-Check.pdb`

**YARA rule:**

```
rule thm_demo_rule {
    meta:
        author = "THM: Intro-to-AV-Room"
        description = "Look at how the Yara rule works with ClamAV"
    strings:
        $a = "C:\\Users\\thm\\source\\repos\\AV-Check\\AV-Check\\obj\\Debug\\AV-Check.pdb"
    condition:
        $a
}
```

Scan using the rule:

```
clamscan.exe -d Desktop\Files\thm-demo-1.yara Desktop\Samples
```

Result: `AV-Check.exe` detected, but `notes.txt` also detected (**false positive**) because it contains the same string.

### **Improved YARA Rule (Reduce False Positives)**

Windows executable magic bytes: `MZ` (`4D 5A`), found at offset `0`.

```
rule thm_demo_rule {
    meta:
        author = "THM: Intro-to-AV-Room"
        description = "Look at how the Yara rule works with ClamAV"
    strings:
        $a = "C:\\Users\\thm\\source\\repos\\AV-Check\\AV-Check\\obj\\Debug\\AV-Check.pdb"
        $b = "MZ"
    condition:
        $b at 0 and $a
}
```

```
clamscan.exe -d Desktop\Files\thm-demo-2.yara Desktop\Samples
```

Result: `AV-Check.exe` detected, `notes.txt` no longer flagged.

### **Main Limitation**

Signature-based detection is weak against modified binaries because changing the file changes its hash/signature. Malware can evade detection when its known patterns are altered.


### --------------------------------- Other Detection Techniques -----------------------------------

## **Other Detection Techniques**

### **1. Dynamic Detection**

Detects malware **while it runs**, using:

- **API Monitoring/Hooks:** Monitor Windows API calls and application behavior.
- **Sandboxing:** Runs malware in an isolated virtual environment to observe its behavior.

Sandbox limitations: Limited execution time, malware may detect virtual environments or delay execution to avoid analysis.

<img width="826" height="316" alt="image" src="https://github.com/user-attachments/assets/880590c2-4232-4223-85b0-031328f5de11" />

Reference: [Sandbox Evasion](https://tryhackme.com/room/sandboxevasion?utm_source=chatgpt.com)

### **2. Heuristic & Behavioral Detection**

- **Static Heuristic:** Decompiles/extracts code and compares it with known malware code in a heuristic database. If similarity exceeds a threshold, flags it.
- **Dynamic Heuristic:** Uses predefined behavioral rules based on malware analysis.

<img width="819" height="307" alt="image" src="https://github.com/user-attachments/assets/65387207-6692-42ae-8f12-bac1f42a6999" />

Examples of behavioral rules: process accessing `LSASS.exe` (contains NTLM hashes, Kerberos tickets, etc.), process opening a listening port to receive commands from a C2 server.

### **3. AV Scanning Flow**

<img width="783" height="534" alt="image" src="https://github.com/user-attachments/assets/6e3eb54a-5de7-4a7b-9279-e2e6fa03ab9a" />

Suspicious file → Identify file type → Extract archive → Identify file type → PE parsing → Check packing → Unpack if needed → AV engine analyzes collected data → Verdict (malicious/clean).

| Detection  | How it works                                      |
| ---------- | ------------------------------------------------- |
| Static     | Matches known signatures/patterns                 |
| Dynamic    | Monitors behavior during execution                |
| Heuristic  | Detects suspicious code/features                  |
| Behavioral | Detects suspicious actions using predefined rules |


### --------------------------------- AV Testing and Fingerprinting -----------------------------------

## **AV Testing & Fingerprinting**

### **AV Testing**

AV testing platforms scan suspicious files using multiple detection engines to evaluate detection results.

- **VirusTotal:** Scans files using 70+ AV engines, signatures, binary analysis, behavioral analysis, API calls, and sandbox simulation.
- **Important:** VirusTotal shares uploaded files/results with AV vendors, which may expose payloads used in engagements.

Alternatives mentioned: AntiscanMe (6 free scans/day), Jotti's malware scan.

### **AV Fingerprinting**

Identifying installed AV/EDR products using static artifacts, such as service names, process names, DLLs, registry keys, directories, and files.

| Antivirus          | Service                               | Process                                |
| ------------------ | ------------------------------------- | -------------------------------------- |
| Microsoft Defender | `WinDefend`                           | `MsMpEng.exe`                          |
| Trend Micro        | `TMBMSRV`                             | `TMBMSRV.exe`                          |
| Avira              | `AntivirService`, `Avira.ServiceHost` | `avguard.exe`, `Avira.ServiceHost.exe` |
| Bitdefender        | `VSSERV`                              | `bdagent.exe`, `vsserv.exe`            |
| Kaspersky          | `AVP<Version #>`                      | `avp.exe`, `ksde.exe`                  |
| AVG                | `AVG Antivirus`                       | `AVGSvc.exe`                           |
| Norton             | `Norton Security`                     | `NortonSecurity.exe`                   |
| McAfee             | `McAPExe`, `Mfemms`                   | `MCAPExe.exe`, `mfemms.exe`            |
| Panda              | `PavPrSvr`                            | `PavPrSvr.exe`                         |
| Avast              | `Avast Antivirus`                     | `afwServ.exe`, `AvastSvc.exe`          |

### **SharpEDRChecker**

Public C# tool that fingerprints security products using processes, file metadata, loaded DLLs, Registry keys, services, directories, and files.

Example findings:

- Directories: Windows Defender directories
- Services: `WinDefend`, `WdNisSvc`, `Sense`, `wscsvc`

### **C# AV Check**

The provided C# program uses **WMI (**`Win32_Process`**)** to enumerate running processes and compares their names against a predefined AV process list.

- Match found → AV detected
- No match → AV not found

WMI query: `select * from win32_process`

### **False Positives**

VirusTotal may flag legitimate programs as malicious due to poorly implemented ML classifiers or detection rules.

The `AV-Check.exe` program was flagged by some vendors despite being legitimate. Detection could be triggered by its API calls, process enumeration, or behavioral rules. Different builds (x86/x64) may produce different detection results.

**Remember:** AV detection results aren't always accurate, VirusTotal results can vary, and uploaded files may be shared with AV vendors.




# AV Evasion: Shellcode

### --------------------------------- PE Structure -----------------------------------

## **PE Structure (Portable Executable)**

**PE:** Windows executable file format used by `.exe`, `.dll`, and object files. It organizes code, data, resources, imports, and metadata so the Windows loader can load and execute a program (x86/x64).

<img width="762" height="555" alt="image" src="https://github.com/user-attachments/assets/09a012d9-f4c7-4db5-be42-087fcb543d48" />

### **PE Sections**

| Section  | Purpose                                             |
| -------- | --------------------------------------------------- |
| `.text`  | Executable code                                     |
| `.data`  | Initialized variables                               |
| `.bss`   | Uninitialized variables                             |
| `.rdata` | Read-only data                                      |
| `.edata` | Exported objects/functions                          |
| `.idata` | Imported DLLs/functions                             |
| `.reloc` | Relocation information                              |
| `.rsrc`  | Resources (icons, images, embedded files, manifest) |

### **Windows Loader Process**

1. Parses headers (DOS, Windows, Optional), checks `MZ`, signatures, architecture, timestamp.
2. Reads the section table and number of sections.
3. Maps sections into memory using `ImageBase` and `EntryPoint`.
4. Loads imports (DLLs and external functions).
5. Finds the `EntryPoint` and starts execution.

**Important terms:** `MZ` = EXE magic signature, `ImageBase` = preferred memory loading address, `RVA` = address relative to image base, `EntryPoint` = starting execution location.

### **Why PE Matters?**

PE headers and sections are used by malware analysts and AV to analyze binaries. PE knowledge is important for understanding packing/unpacking and malware analysis.

### **PE-Bear**

GUI tool for analyzing PE files, headers, sections, imports, and other binary details. Load a file using **File → Load PEs** or `Ctrl + O`.

### **Remember**

Shellcode/data placement can vary depending on how the program stores data, such as `.text`, `.data`, `.rsrc`, or custom sections.

**Answer:** PE headers are data type `STRUCT`.


### --------------------------------- Introduction to Shellcode -----------------------------------

## **Introduction to Shellcode**

**Shellcode:** Crafted machine-code instructions (opcodes) that execute additional code inside a vulnerable program/process, often used to execute commands or establish a shell.

Written in Assembly and converted into hexadecimal machine code. Requires knowledge of x86/x64, Assembly, C, Linux/Windows internals.

### **Syscalls (64-bit Linux)**

A syscall requests the kernel to perform an operation.

| `rax`  | Syscall     | Parameters                                       |
| ------ | ----------- | ------------------------------------------------ |
| `0x1`  | `sys_write` | `rdi` = FD, `rsi` = string address, `rdx` = size |
| `0x3c` | `sys_exit`  | `rdi` = exit code                                |

`sys_write`: `rdi=1` → STDOUT, `rsi` → message address, `rdx` → message length. `sys_exit`: `rdi=0` → successful exit.

### **Assembly Example**

`thm.asm` prints `THM, Rocks!` using `sys_write` and exits using `sys_exit`.

- `jmp MESSAGE` → jumps to message
- `call GOBACK` → pushes message address onto stack
- `pop rsi` → retrieves message address
- `syscall` → executes kernel function
- `db` → stores message bytes, `0dh, 0ah` = `\r\n`

### **Compile & Execute**

```
nasm -f elf64 thm.asm
ld thm.o -o thm
./thm
```

### **Extract Shellcode**

Disassemble binary:

```
objdump -d thm
```

Extract `.text` section:

```
objcopy -j .text -O binary thm thm.text
```

Convert binary to hex:

```
xxd -i thm.text
```

Output: `new_text[]` contains the machine-code bytes, `new_text_len = 50`.

### **Shellcode Execution**

Shellcode bytes can be stored in a C array and executed as code.

```
(*(void(*)())message)();
```

This casts `message` into a function pointer and executes it.

Compile:

```
gcc -g -Wall -z execstack thm.c -o thmx
./thmx
```

`-z execstack` enables executable stack memory (disables NX protection for the stack).

**Remember:** Assembly → machine code → extract `.text` → convert to hex → shellcode.


### --------------------------------- Generate Shellcode -----------------------------------

## **Generate Shellcode**

### **Public Tools (MSFvenom)**

- `msfvenom` generates shellcode for a specific **platform, architecture, and output format**.
- Advantage: no need to write shellcode manually or know Assembly deeply.
- Disadvantage: generated shellcode is well-known and easily detected by AV.

### **Shellcode formats**

- `-f c` → C array format
- `-f raw` → binary `.bin` format
- `xxd -i` → convert binary shellcode into a C array

### **Key concepts**

- **Shellcode injection:** injects code into a process/thread and changes execution flow to run it.
- **Dropper:** main executable that contains or delivers shellcode.
- **AV detection:** public/generated payloads are commonly recognizable by signatures.

### **Raw shellcode conversion**

```
file /tmp/example.bin
xxd -i /tmp/example.bin
```

- `.bin` stores raw binary data.
- `xxd -i` converts binary bytes into a C array.
- Generated array contains the shellcode and its length (`193 bytes` in the example).

### **Important**

The same shellcode can be represented in different formats, such as C arrays or raw binary. The underlying bytes remain the same when converted correctly.

**Remember:** `MSFvenom → shellcode → C/raw format → executable memory → execution`


### --------------------------------- Staged Payloads -----------------------------------

## **Staged vs Stageless Payloads**

### **Stageless Payload**

- Final shellcode is **embedded inside the executable**.
- Runs without downloading additional payloads.
- One-step execution.

### **Staged Payload**

- Uses multiple stages.
- **Stage 0 (stager):** small initial code, retrieves the final payload.
- **Stage 1:** final shellcode, loaded into memory and executed.
- Usually involves an additional network connection.

### **Comparison**

| Stageless                       | Staged                                        |
| ------------------------------- | --------------------------------------------- |
| Everything included             | Final payload downloaded later                |
| Larger file                     | Smaller initial file                          |
| No additional download required | Requires network connection                   |
| Payload already embedded        | Final payload isn't embedded                  |
| Self-contained                  | Reuses the same stager for different payloads |

### **Metasploit Naming**

| Payload                               | Type      |
| ------------------------------------- | --------- |
| `windows/x64/shell_reverse_tcp`       | Stageless |
| `windows/x64/shell/reverse_tcp`       | Staged    |
| `windows/x64/meterpreter_reverse_tcp` | Stageless |
| `windows/x64/meterpreter/reverse_tcp` | Staged    |

**Remember:** `_` generally indicates stageless, while `/` indicates staged.

### **Stager APIs**

| API                     | Purpose                     |
| ----------------------- | --------------------------- |
| `VirtualAlloc()`        | Allocates memory            |
| `CreateThread()`        | Creates a thread            |
| `WaitForSingleObject()` | Waits for thread completion |
| `Marshal.Copy()`        | Copies data into memory     |
| `WebClient()`           | Downloads data              |
| `DownloadData()`        | Retrieves binary data       |

### **Stager Concept**

- Downloads shellcode from a URL.
- Allocates memory for it.
- Copies shellcode into memory.
- Executes it in a thread.
- Waits for the thread to finish.

### **Main Differences**

- **Stageless:** useful when the target has limited/no network access after delivery.
- **Staged:** smaller initial payload, final payload delivered separately and kept in memory.
- Staged payloads can reduce the initial file footprint, but neither type is inherently better—it depends on the environment.


### --------------------------------- Introduction to Encoding and Encryption -----------------------------------

## **Encoding vs Encryption**

### **Encoding**

- Converts data into another format using an encoding algorithm.
- Used for storage, transmission, processing, and file conversion.
- **No secret key required**; can be reversed using the same encoding method.
- AV can decode encoded strings and inspect the original content.

**Example:** Hex → Base64 (multiple encoding layers).

### **Encryption**

- Converts **plaintext → ciphertext** to protect data.
- Requires an encryption algorithm and key to decrypt.
- Used for secure storage, transmission, and confidentiality.
- Uses shared-key (symmetric) or public/private-key (asymmetric) encryption.

### **Why used in AV research?**

- Public tools and shellcode may be recognized by AV through static or dynamic detection.
- Encoding/encryption changes how data is represented.
- AV can still analyze or decode suspicious content.

| Encoding            | Encryption                    |
| ------------------- | ----------------------------- |
| Changes data format | Protects data confidentiality |
| No secret key       | Requires key                  |
| Easily reversible   | Requires decryption           |
| Base64, Hex         | AES, RSA                      |

**Remember:** Encoding changes representation; encryption protects data using a key.


### --------------------------------- Shellcode Encoding and Encryption -----------------------------------

## **Shellcode Encoding & Encryption**

### **MSFvenom Encoding**

- Metasploit provides encoders, but AV vendors recognize their patterns.
- Multiple encoding iterations do **not guarantee AV bypass**.
- Example encoder: `x86/shikata_ga_nai`.
- Encoding changes representation, but AV may still detect the payload.

### **MSFvenom Encryption**

Available encryption formats:

`aes256`, `base64`, `rc4`, `xor`

- Encryption transforms payload data using a key.
- Standard MSFvenom-encrypted payloads can still be detected by AV.

### **Custom Encoding**

The room demonstrates combining **XOR + Base64**.

- XOR → transforms bytes using a key.
- Base64 → converts encoded bytes into text.
- Decoding must happen in reverse order: **Base64 decode → XOR**.
- Encoder and decoder must use the same key.

### **Key takeaway**

| Method                       | Result                                                                |
| ---------------------------- | --------------------------------------------------------------------- |
| Standard encoding            | Often detected                                                        |
| Multiple encoding iterations | Not guaranteed to evade AV                                            |
| Standard encryption          | Can still be detected                                                 |
| Custom encoding              | Changes payload representation, but doesn't guarantee undetectability |

**Remember:** Encoding/encryption alone does not make malware undetectable. AV can also analyze behavior, memory, API calls and execution patterns.


### --------------------------------- Packers -----------------------------------

## **Packers**

- **Packer:** transforms a program's structure while preserving its functionality.
- Main purposes: compression, reverse-engineering protection, obfuscation.
- Common packers: `UPX`, `MPRESS`, `Themida`, `ConfuserEx`.

<img width="1379" height="798" alt="image" src="https://github.com/user-attachments/assets/b935b2bb-5274-4e28-bf67-fe76e6960c5f" />

### **How packing works**

1. Original code is transformed/packed.
2. An **unpacker stub** is added and becomes the entry point.
3. Stub runs first and unpacks the original code into memory.
4. Execution is redirected to the original code.

### **AV Detection**

- Packed code may avoid file-based signature matches.
- AV may still detect the **unpacker stub**.
- In-memory scanning can detect the code after unpacking.

### **Important**

- Packing does not guarantee undetectability.
- AV may scan memory and monitor API calls.
- Detection behavior varies between AV solutions.

**Remember:** `Packed program → Unpacker stub → Original code restored in memory → Execution`


### --------------------------------- Binders -----------------------------------

## **Binders**

- **Binder:** combines two or more executables into a single executable.
- Main purpose: make a payload appear to be a legitimate program.
- The original program may continue working normally.

### **How it works**

- Combines a legitimate executable with another executable/payload.
- Payload may execute separately (e.g., through another thread).
- The legitimate program continues running.

### **AV Detection**

- Binding alone **does not bypass AV**.
- Original payload signatures remain detectable in the combined executable.
- Packers/encoders may change file characteristics, but do not guarantee evasion.

### **Main purpose**

- Primarily tricks users into believing they are opening a legitimate program.
- Doesn't inherently hide malicious code from antivirus.

**Remember:** `Binder = Multiple executables → One executable`





# Obfuscation Principles

### --------------------------------- Origins of Obfuscation -----------------------------------

## **Origins of Obfuscation**

- **Obfuscation:** Makes code difficult to understand while preserving its functionality.
- **Purpose:** Protect intellectual property (IP), prevent reverse engineering, and make code analysis harder.
- Example: Minecraft uses **ProGuard** to obfuscate/minimize Java classes. Obfuscation maps help modders translate obfuscated classes.

<img width="920" height="596" alt="image" src="https://github.com/user-attachments/assets/625e3a3f-6caf-4f27-9143-073093a5f8b3" />

### **Obfuscation Layers**

| Layer              | Focus                            |
| ------------------ | -------------------------------- |
| Code-Element       | Individual code elements         |
| Software-Component | Individual software components   |
| Inter-Component    | Communication between components |
| Application        | Entire application               |

<img width="1088" height="712" alt="image" src="https://github.com/user-attachments/assets/d25f6d37-24e5-40bb-832c-df8d37d43a4a" />

### **Code-Element Layer**

Sub-layers:

- Obfuscating Layout
- Obfuscating Controls
- Obfuscating Data
- Obfuscating Methods
- Obfuscating Classes

### **Example**

If the goal is to obfuscate code layout without modifying existing code, the taxonomy suggests:

`Code-Element Layer → Obfuscating Layout → Junk Code`

### **Why it matters in security**

Obfuscation can be used to make malware harder to analyze and to interfere with signature-based detection.

**Remember:** Obfuscation changes how code looks or is structured, not what it ultimately does.

- Research paper: [Layered obfuscation taxonomy](https://cybersecurity.springeropen.com/track/pdf/10.1186/s42400-020-00049-3?utm_source=chatgpt.com)
- [ProGuard](https://github.com/Guardsquare/proguard?utm_source=chatgpt.com)


### --------------------------------- Obfuscation's Function for Static Evasion -----------------------------------

## **Obfuscation's Function for Static Evasion**

- **AV & EDR** use static signatures and heuristic detection to identify malware.
- **Obfuscation** changes how data/code appears to make signature detection and analysis harder.

### **Data Obfuscation Methods**

| Method                 | Purpose                                                         |
| ---------------------- | --------------------------------------------------------------- |
| Array Transformation   | Split, merge, fold, or flatten arrays                           |
| Data Encoding          | Transform data using mathematical functions or ciphers          |
| Data Procedurization   | Replace static data with procedure/function calls               |
| Data Splitting/Merging | Distribute one variable's information across multiple variables |

**Focus:** Data Splitting/Merging.

- Static signatures are based on identifiable patterns.
- Obfuscation changes those patterns, making static detection less reliable.

**References:**

- [Layered Obfuscation Taxonomy](https://cybersecurity.springeropen.com/articles/10.1186/s42400-020-00049-3?utm_source=chatgpt.com)
- [Encoding/Packing/Binder/Crypters](https://tryhackme.com/room/encodingpackingbindercry?utm_source=chatgpt.com)
- [Signature Evasion](https://tryhackme.com/room/signatureevasion?utm_source=chatgpt.com)


### --------------------------------- Object Concatenation -----------------------------------

## **Object Concatenation**

- **Concatenation:** Combines multiple objects/strings into one (`A + B`).
- Used in obfuscation to alter recognizable static patterns.
- **Non-interpreted characters:** Can affect how strings appear without changing how they are interpreted.

### **Concatenation operators**

| Language   | Operators                           |
| ---------- | ----------------------------------- |
| Python     | `+`                                 |
| PowerShell | `+`, `,`, `$`, or none              |
| C#         | `+`, `String.Join`, `String.Concat` |
| C          | `strcat`                            |
| C++        | `+`, `append`                       |

### **String Obfuscation Concepts**

| Technique   | Purpose                     |
| ----------- | --------------------------- |
| Breaks      | Split strings into parts    |
| Reorders    | Rearrange string components |
| Whitespace  | Add ignored whitespace      |
| Ticks       | Add ignored tick characters |
| Random Case | Change capitalization       |

### **Key takeaway**

Static signatures look for recognizable strings or patterns. Concatenation changes how strings appear in source code, while the program may retain the same behavior.

- [Layered Obfuscation Taxonomy](https://cybersecurity.springeropen.com/articles/10.1186/s42400-020-00049-3?utm_source=chatgpt.com)
- [Signature Evasion – THM](https://tryhackme.com/room/signatureevasion?utm_source=chatgpt.com)


### --------------------------------- Obfuscation's Function for Analysis Deception -----------------------------------

## **Obfuscation's Function for Analysis Deception**

Obfuscation makes code harder for analysts and reverse engineers to understand, even if it passes automated detection.

### **Obfuscation Methods**

| Method                      | Purpose                                                      |
| --------------------------- | ------------------------------------------------------------ |
| Junk Code                   | Adds non-functional code/stubs                               |
| Separation of Related Code  | Separates related instructions to make code harder to follow |
| Stripping Redundant Symbols | Removes debug information and symbol tables                  |
| Meaningless Identifiers     | Replaces meaningful names with meaningless ones              |
| Implicit Controls           | Changes explicit control instructions into implicit ones     |
| Dispatcher-based Controls   | Determines which code block executes at runtime              |
| Probabilistic Control Flows | Uses equivalent logic with different structures              |
| Bogus Control Flows         | Adds code paths that never execute                           |

**Key takeaway:** These techniques make code harder to read, analyze, and reverse engineer.

**References:**

- [Malware Analysis Module](https://tryhackme.com/module/malware-analysis?utm_source=chatgpt.com)
- [Layered Obfuscation Taxonomy](https://cybersecurity.springeropen.com/articles/10.1186/s42400-020-00049-3?utm_source=chatgpt.com)
- [Sandbox Evasion](https://tryhackme.com/room/sandboxevasion?utm_source=chatgpt.com)


### --------------------------------- Code Flow and Logic -----------------------------------

## **Code Flow and Logic**

- **Control Flow:** Defines the order in which a program executes.
- **Logic:** Determines which execution path the program follows.

### **Logic Statements**

| Statement     | Purpose                             |
| ------------- | ----------------------------------- |
| `if/else`     | Executes code based on a condition  |
| `try/catch`   | Handles errors/exceptions           |
| `switch/case` | Selects between multiple conditions |
| `for/while`   | Repeats code based on conditions    |

### **Example**

```
x = 10if x > 7:    print("This executes")else:    print("This is ignored")
```

- **CFG (Control Flow Graph):** Visual representation of possible execution paths.
- Logic can change a program's control flow.
- Complex or confusing logic makes code harder for analysts to understand.

**Remember:** Logic determines *where the program goes next*.


### --------------------------------- Arbitrary Control Flow Patterns -----------------------------------

## **Arbitrary Control Flow Patterns**

- **Predicate:** A condition that returns `True` or `False`.
- **Opaque Predicate:** A condition whose result is known to the author but difficult for an analyst to determine.
- Used to make program logic and execution paths harder to understand.

### **Opaque Predicates**

- Can introduce confusing or unnecessary logic.
- Related to **Bogus Control Flow** and **Probabilistic Control Flow**.
- Makes reverse engineering and CFG analysis more difficult.

### **Collatz Conjecture**

For any positive integer:

- If even → divide by 2.
- If odd → multiply by 3 and add 1.
- Repeated operations are conjectured to eventually reach `1`.

Example condition: If the input is a positive integer greater than 1, the Collatz sequence eventually reaches `1`.

**Key takeaway:** Opaque predicates add complex conditions to make control flow difficult to analyze, while the actual program behavior may remain unchanged.

- [Opaque Predicate Research](https://etda.libraries.psu.edu/files/final_submissions/17513?utm_source=chatgpt.com)
- [Collatz Problem](https://mathworld.wolfram.com/CollatzProblem.html?utm_source=chatgpt.com)


### --------------------------------- Protecting and Stripping Identifiable Information -----------------------------------

## **Protecting and Stripping Identifiable Information**

Identifiable information helps analysts understand a program's purpose. Removing it makes analysis and reverse engineering harder.

### **1. Object Names**

- Meaningful variable, function, and object names reveal functionality.
- **Interpreted languages:** identifiers generally need to be changed throughout the code.
- **Compiled languages:** identifiable strings and symbols are especially important.

### **2. Code Structure**

- Code structure can reveal malicious functions and signatures.
- Junk code and reordered code make analysis harder.
- Related functions/calls may be separated, making their relationship harder to identify.

### **3. File & Compilation Properties**

- Debug builds may expose symbols, variables, function names, and entry points.
- Symbol/debug information can help reverse engineers.
- `Debug` vs `Release` compilation affects included information.
- `strip` removes symbol information from compiled binaries.

| Concept                    | Purpose                                    |
| -------------------------- | ------------------------------------------ |
| Meaningless Identifiers    | Hide meaningful object names               |
| Junk Code                  | Add unnecessary code                       |
| Code Reordering            | Change code structure                      |
| Separation of Related Code | Make related operations harder to identify |
| Symbol Stripping           | Remove debugging/symbol information        |

**Key takeaway:** Analysts can use strings, identifiers, code structure, symbols, and compilation metadata to understand a program.





# Signature Evasion

### --------------------------------- Signature Identification -----------------------------------

## **Signature Identification**

- **Signature:** A pattern used by AV to identify suspicious/malicious files.
- Signatures may match specific bytes or patterns within a compiled binary.
- Binary signatures can be located using iterative analysis and a hex viewer.
- Binary data is not necessarily human-readable.

### **Key concept**

Binary files contain bytes. Signature locations can be identified by examining file data and understanding which regions trigger detection.

**Remember:** Signature identification helps analysts understand *what* a detection matches.


### --------------------------------- Automating Signature Identification -----------------------------------

## **Automating Signature Identification**

Manual signature identification is slow, so tools automate detection analysis.

### **Tools**

| Tool             | Purpose                                                         |
| ---------------- | --------------------------------------------------------------- |
| Find-AVSignature | Splits binary ranges to identify possible signatures            |
| DefenderCheck    | Identifies detected regions in files                            |
| ThreatCheck      | Uses AV engines to locate detected byte regions                 |
| AMSITrigger      | Identifies suspicious sections in PowerShell scripts using AMSI |

### **Find-AVSignature**

- Splits binary data into intervals.
- Requires choosing an interval size.
- Primarily checks file contents/signatures, not full AV behavior.

### **ThreatCheck**

- Analyzes files using detection engines.
- Supports **Defender** and **AMSI** engines.
- Can report detected byte regions and offsets.
- May produce false positives.

### **AMSITrigger**

- Scans PowerShell scripts through AMSI.
- Identifies sections that trigger detection.
- Supports different output formats and line numbers.

**Key takeaway:** Automated tools help analysts understand where detections originate, but results may be incomplete or inaccurate.

Links:

- [Find-AVSignature](https://github.com/PowerShellMafia/PowerSploit/blob/master/AntivirusBypass/Find-AVSignature.ps1?utm_source=chatgpt.com)
- [DefenderCheck](https://github.com/matterpreter/DefenderCheck?utm_source=chatgpt.com)
- [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck?utm_source=chatgpt.com)
- [AMSITrigger](https://github.com/RythmStick/AMSITrigger?utm_source=chatgpt.com)


### --------------------------------- Static Code-Based Signatures -----------------------------------

## **Static Code-Based Signatures**

### **Obfuscating Methods & Classes**

| Method                        | Purpose                                     |
| ----------------------------- | ------------------------------------------- |
| Method Proxy                  | Replace method/object with a proxy          |
| Method Scattering/Aggregation | Split or combine methods                    |
| Method Clone                  | Duplicate methods                           |
| Class Hierarchy Flattening    | Use interfaces/proxies                      |
| Class Splitting/Coalescing    | Move variables/instructions between classes |
| Dropping Modifiers            | Remove access modifiers                     |

### **Main concepts**

- **Splitting/Merging:** Split or combine methods, classes, or objects while maintaining functionality.
- **Removing/Obscuring identifiers:** Hide recognizable strings, names, or information used by signatures.
- **Method Proxy:** Replace identifiable methods or strings with alternative references.
- Static signatures target recognizable code/string patterns.

### **Example concepts**

- `StringBuilder` → combine multiple string parts instead of one recognizable string.
- `wdigest.dll` → identifiable string example from Mimikatz.

**Key:** Obfuscation changes code appearance/structure, making static signature matching and analysis harder.

### **THM Challenge**

- Tool: `AMSITrigger`
- File: `challenge-1.ps1`
- Target: `challenge-1.html`
- Goal: Understand how static signatures identify recognizable code patterns.

I can help you understand the concepts behind the PowerShell snippet and its AMSI detection, but I can't provide a modified version designed to bypass AMSI and disable security scanning.


### --------------------------------- Static Property-Based Signatures -----------------------------------

## **Static Property-Based Signatures**

Static detection can use file properties, not just strings or code signatures.

### **File Properties**

- **Hash:** Identifies a file; any modification changes its hash.
- **Entropy:** Measures data randomness; high entropy can indicate encryption or obfuscation.
- Other indicators: author, filename, PE headers, module properties.

### **File Hash**

- Hash/checksum is used to identify files and verify integrity.
- Different file content → different hash.
- Source available → modify code and recompile.
- Closed-source/signed files → modifications can affect functionality and signatures.

Bit-flipping concept: Changing bits modifies file content and therefore its hash, but may also break functionality or invalidate its digital signature.

### **Entropy (Shannon)**

Entropy measures randomness in file data. EDRs use it as one indicator of potentially suspicious content.

| Content                        | Entropy |
| ------------------------------ | ------- |
| English paragraph              | 4.587   |
| Script with random identifiers | 5.341   |
| Potentially suspicious         | \~6.8+  |

- Random identifiers can increase entropy.
- Encrypted/compressed data often has higher entropy.
- High entropy alone does **not** prove malware.

**Key:** Hash identifies a file; entropy measures randomness. Both are supporting indicators, not standalone proof of malicious activity.

### **Links**

- [IBM – File Entropy](https://www.ibm.com/docs/en/qsip/7.4?topic=content-analyzing-files-embedded-malicious-activity\&utm_source=chatgpt.com)
- [CyberChef – Shannon Entropy](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com#recipe=Entropy\(%27Shannon%20scale%27\))
- [EDR Assessment Paper](https://www.mdpi.com/2624-800X/1/3/21/pdf?utm_source=chatgpt.com)


### --------------------------------- Behavioral Signatures -----------------------------------

## **Behavioral Signatures**

- AV/EDR monitors **program behavior**, not just static strings or hashes.
- Common detection methods:
  - **Import monitoring** – examines imported API functions (IAT).
  - **API hooking** – monitors/intercepts function calls.

### **IAT (Import Address Table)**

- Stores imported API function addresses (filled by the Windows loader at runtime).
- **ASLR** makes function addresses dynamic.
- PE imports can reveal what a program does.

### **Dynamic Loading**

Instead of normal imports, programs can obtain API addresses at runtime.

Key concepts:

- Function pointer
- Module handle
- API address
- Function structure/signature

**Key:** Dynamic loading changes how API imports appear, but behavior can still be monitored through hooks and other detection methods.

### **Limitations**

- API calls can still reveal suspicious behavior.
- `LoadLibraryA` and `GetProcAddress` may themselves appear in the IAT.
- API hooking monitors calls at runtime.





# Bypassing UAC

### --------------------------------- User Account Control (UAC) -----------------------------------

## **User Account Control (UAC)**

### **What is UAC?**

UAC prevents programs from automatically running with Administrator privileges. Even administrators normally run applications with a **filtered token** until elevation is requested.

### **Integrity Levels (IL)**

| Level  | Purpose                                           |
| ------ | ------------------------------------------------- |
| Low    | Restricted processes (e.g., Internet-facing apps) |
| Medium | Standard users & filtered administrators          |
| High   | Elevated administrators                           |
| System | Windows system processes                          |

**MIC** controls access based on integrity levels and takes precedence over DACL permissions.

### **Filtered Tokens**

- **Standard user:** One Medium IL token.
- **Administrator:** Two tokens:
  - Filtered token → Medium IL, used normally.
  - Elevated token → High IL, used after elevation.

### **UAC Settings**

1. Always notify
2. Notify when programs make changes
3. Same as above, without secure desktop
4. Never notify → administrators run with High IL

Default: **Notify when programs try to make changes.**

### **UAC Internals**

1. User requests *Run as administrator*.
2. `ShellExecute` with `runas` verb.
3. Request forwarded to **Appinfo** service.
4. Application manifest checked.
5. `consent.exe` displays prompt on secure desktop.
6. After consent, Appinfo launches using the elevated token.

### **UAC Bypass – Core Concept**

- An administrator's remote shell may still have **Medium IL**.
- Administrator group may appear as **Deny only** in `whoami /groups`.
- Administrative actions may fail with `Access Denied` (`System error 5`).
- UAC is **not considered a security boundary** by Microsoft.
- Bypass techniques generally abuse a process already running at **High IL**.

**Key:** Administrator membership ≠ elevated privileges. Medium IL = filtered token; High IL = elevated token.


### --------------------------------- UAC: GUI based bypasses -----------------------------------

## **UAC: GUI-Based Bypasses**

### **Core concept**

Some Windows applications **auto-elevate** to High Integrity Level (IL) without showing a UAC prompt. Child processes may inherit the elevated token.

### **Case 1:&#x20;**`msconfig`

- `msconfig` auto-elevates to **High IL**.
- Its **Tools** tab can launch a command prompt.
- The spawned shell inherits the elevated token.

Key: Auto-elevated process → child process → High IL.

### **Case 2:&#x20;**`azman.msc`

- `azman.msc` auto-elevates through `mmc.exe`.
- Its Help feature can launch another process.
- Process tree shows the High IL token being inherited through child processes.

### **Important Terms**

| Term              | Meaning                                                      |
| ----------------- | ------------------------------------------------------------ |
| Auto-elevation    | Certain Windows programs elevate without a normal UAC prompt |
| High IL           | Elevated integrity level                                     |
| `mmc.exe`         | Microsoft Management Console                                 |
| Token inheritance | Child process may inherit parent security context            |

### **Lab Environment**

```
xfreerdp /v:MACHINE_IP /u:attacker /p:Password321
```

**Key takeaway:** GUI-based UAC bypasses rely on auto-elevated applications and their ability to spawn child processes with inherited elevated privileges.


### --------------------------------- UAC: Auto-Elevating Processes -----------------------------------

## **UAC: Auto-Elevating Processes**

### **Auto-Elevation Requirements**

An application may auto-elevate if:

- Signed by **Windows Publisher**
- Located in a trusted directory (`System32`, `Program Files`)
- EXE manifest contains `autoElevate=true`
- Certain `.msc` snap-ins can trigger `mmc.exe` auto-elevation.
- Some executables are allowlisted; COM objects can also request elevation.

Check manifest using `sigcheck`:

```
sigcheck64.exe -m c:\windows\system32\msconfig.exe
```

### **Fodhelper**

- `fodhelper.exe` manages Windows optional features.
- It is an auto-elevating executable.
- Can run from a Medium IL context.
- Abuse involves Windows registry file associations (**ProgID**).

### **Registry Concepts**

| Registry                | Purpose                    |
| ----------------------- | -------------------------- |
| `HKLM\Software\Classes` | System-wide associations   |
| `HKCU\Software\Classes` | User-specific associations |
| `HKCR`                  | Merged view of HKLM + HKCU |

HKCU associations take priority over HKLM associations.

### **Integrity Verification**

```
whoami
whoami /groups
```

| IL     | SID            |
| ------ | -------------- |
| Medium | `S-1-16-8192`  |
| High   | `S-1-16-12288` |

- Administrators at Medium IL → filtered token.
- High IL → elevated token.

### **Key takeaway**

Auto-elevation + registry association behavior can create UAC bypass opportunities. **UAC is not a security boundary**, and auto-elevating processes should be carefully monitored.


### --------------------------------- UAC: Improving the Fodhelper Exploit to Bypass Windows Defender -----------------------------------

## **UAC: Improving Fodhelper Exploit**

### **Windows Defender Detection**

- Defender monitors suspicious registry modifications related to UAC bypass.
- It detects the `ms-settings` registry modification and may delete the offending value.
- There can be a delay between detection and removal.

### **Detection Behavior**

- Defender may detect the exploit through specific registry patterns.
- Execution timing can affect the result, but this behavior is unreliable.
- Detection is not necessarily based on the payload alone; registry activity can trigger alerts.

### **ProgID & CurVer**

- **ProgID:** Registry identifier that associates a file type/object with an application.
- **CurVer:** Points to the current/default version of a ProgID.
- `HKCU\Software\Classes` contains user-level associations.
- Registry associations can influence how Windows resolves application commands.

### **Key Takeaways**

- Defender can detect UAC bypass attempts through **registry activity and behavior**, not just executable signatures.
- Changing implementation details does not necessarily prevent behavioral detection.
- Race conditions are unreliable and should not be treated as consistent behavior.
- Detection can depend on implementation, execution context, registry changes and security products.

**Remember:** UAC bypass techniques abuse auto-elevated processes and Windows configuration behavior; Defender can detect these actions even when the payload itself changes.


### --------------------------------- UAC: Environment Variable Expansion -----------------------------------

## **UAC: Environment Variable Expansion**

### **Always Notify**

- **Always Notify** blocks common auto-elevation UAC bypasses (e.g., Fodhelper).
- Scheduled tasks may still run without a UAC prompt.
- Tasks using **Run with highest privileges** use the highest token available to the caller.
- Admin → High IL; standard user → Medium IL.

### **Disk Cleanup Scheduled Task**

- Task: `\Microsoft\Windows\DiskCleanup\SilentCleanup`
- Executable: `cleanmgr.exe`
- Can run on-demand.
- Uses environment variables (`%windir%`, `%systemdrive%`), creating potential security risks if improperly handled.

### **Environment Variable Expansion**

- Windows expands environment variables when executing commands.
- User environment variables can influence command execution if a task relies on them.
- `HKCU\Environment` stores user environment variables.

### **Key Takeaways**

- Scheduled tasks can behave differently from normal UAC elevation.
- **Highest privileges available to the caller ≠ automatic administrator privileges.**
- Environment-variable expansion can introduce command injection risks.
- Always-Notify UAC doesn't eliminate every privilege-escalation risk.


### --------------------------------- Automated Exploitation -----------------------------------

## **Automated Exploitation — UACME**

- **UACME**: Open-source tool for testing Windows UAC bypass techniques.
- **Akagi**: UACME component that executes different bypass methods using method IDs.
- Useful for testing and studying UAC bypasses without manually implementing each technique.

| Method ID | Technique                           |
| --------- | ----------------------------------- |
| 33        | Fodhelper.exe                       |
| 34        | DiskCleanup scheduled task          |
| 70        | Fodhelper using CurVer registry key |

**Key takeaway:** UACME automates known UAC bypass techniques. Detection and success depend on Windows version, configuration, and security controls.

Link: [https://github.com/hfiref0x/UACME](https://github.com/hfiref0x/UACME?utm_source=chatgpt.com)




# Runtime Detection Evasion

### --------------------------------- Runtime Detections -----------------------------------

### **Runtime Detections**

- **Runtime detection:** Scans code before/during execution using signatures, heuristics and behavior; suspicious code may be blocked, quarantined or deleted.
- **CLR:** .NET Common Language Runtime | **DLR:** Dynamic Language Runtime.
- Unlike traditional file scanning, runtime detection can inspect memory and runtime activity; AV can also use it to monitor API calls/hooks.
- **AMSI (Anti-Malware Scan Interface):** Built-in Windows interface that allows AV products to scan runtime content.

**Link:** [AMSI Documentation](https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal?utm_source=chatgpt.com)


### --------------------------------- AMSI Overview -----------------------------------

### **AMSI Overview**

- **AMSI:** Windows interface allowing applications to integrate with antivirus products and scan scripts/content before execution.
- Detects malicious content using scan verdicts; malicious code may be blocked.
- **Response codes:**

| Code          | Meaning          |
| ------------- | ---------------- |
| `0`           | CLEAN            |
| `1`           | NOT_DETECTED     |
| `16384–20479` | BLOCKED_BY_ADMIN |
| `32768`       | DETECTED         |

- **Integrations:** UAC, PowerShell, WScript/CScript, JavaScript, VBScript, Office VBA.
- **Detection:** Scans runtime/script content, including obfuscated and fileless threats—not just files on disk.

**Link:** [Microsoft AMSI Documentation](https://docs.microsoft.com/en-us/windows/win32/amsi/?utm_source=chatgpt.com)


### --------------------------------- AMSI Instrumentation -----------------------------------

### **AMSI Instrumentation**

- AMSI is an **interface**, not an AV itself; registered AV providers scan submitted content.
- PowerShell instrumentation occurs through `System.Management.Automation.dll` (`CompiledScriptBlock.cs`).
- AMSI scans script content and suspicious behavior before execution; malicious content can trigger a `ParseException`.
- **Main API:** `AmsiScanBuffer` (Win32 layer).
- AMSI can be integrated by third-party AV products and other applications.

<img width="936" height="521" alt="image" src="https://github.com/user-attachments/assets/e598cdc8-1404-4868-b3db-6a8e807eeae5" />

<img width="981" height="181" alt="image" src="https://github.com/user-attachments/assets/b03a619c-3c51-4eab-b999-4f523bb60eed" />

**Links:**

- [AMSI ScanBuffer API](https://learn.microsoft.com/en-us/windows/win32/api/amsi/nf-amsi-amsiscanbuffer?utm_source=chatgpt.com)
- [AMSI Functions](https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-functions?utm_source=chatgpt.com)
- [AMSI Stream Interface](https://learn.microsoft.com/en-us/windows/win32/api/amsi/nn-amsi-iamsistream?utm_source=chatgpt.com)
- [InsecurePowerShell](https://github.com/PowerShell/PowerShell/compare/master...cobbr:master?utm_source=chatgpt.com)


### --------------------------------- PowerShell Downgrade -----------------------------------

### **PowerShell Downgrade**

- **PowerShell downgrade:** Runs an older PowerShell engine to lose newer security features.
- PowerShell 2.0 lacks security features introduced in later versions, including AMSI integration.
- **Detection:** Monitor PowerShell version usage and suspicious process launches.
- **Mitigation:** Remove PowerShell 2.0 and block its use through application control.

**Tool:** [Unicorn](https://github.com/trustedsec/unicorn?utm_source=chatgpt.com) — PowerShell payload generation tool.


### --------------------------------- PowerShell Reflection -----------------------------------

### **PowerShell Reflection**

- **Reflection:** Allows PowerShell to inspect and interact with .NET assemblies, types, fields and methods at runtime.
- Can access non-public (private/internal) members using reflection.
- `System.Management.Automation` contains PowerShell's internal AMSI-related components.
- Reflection can be abused to tamper with security-related internal state.
- **Key:** Reflection provides runtime access to .NET internals, so defenders should monitor suspicious reflection activity.


### --------------------------------- Patching AMSI -----------------------------------

### **Patching AMSI**

- AMSI uses `amsi.dll`; its main scanning function is `AmsiScanBuffer`.
- Patching attempts to modify the function's memory so scans return an altered result.
- **Core concepts:** DLL handles, API addresses, memory protection, opcodes, memory patching.
- APIs: `GetModuleHandle`, `GetProcAddress`, `VirtualProtect`, `Marshal.Copy`.
- Detection: Memory modifications to AMSI functions can be detected by AV/EDR.

**References:**

- [BC-Security](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/common/bypasses.py?utm_source=chatgpt.com)
- [RastaMouse AMSI Bypass](https://github.com/rasta-mouse/AmsiScanBufferBypass?utm_source=chatgpt.com)
- [Memory Patching Research](https://rastamouse.me/memory-patching-amsi-bypass/?utm_source=chatgpt.com)
- [GetProcAddress](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress?utm_source=chatgpt.com)
- [GetModuleHandle](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulehandlea?utm_source=chatgpt.com)
- [VirtualProtect](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotect?utm_source=chatgpt.com)
- [Marshal.Copy](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.copy?utm_source=chatgpt.com)


### --------------------------------- Automating for Fun and Profit -----------------------------------

### **Automating AMSI Bypasses**

- **amsi.fail:** Generates randomized, obfuscated PowerShell snippets from known AMSI bypass techniques.
- **AMSITrigger:** Scans scripts through AMSI to identify content triggering detections.
- Detection signatures can target strings, code patterns, and behaviors; obfuscation may change static patterns but doesn't guarantee evasion.

**Links:**

- [amsi.fail](http://amsi.fail?utm_source=chatgpt.com)
- [AMSITrigger](https://github.com/RythmStick/AMSITrigger?utm_source=chatgpt.com)





# Evading Logging and Monitoring

### --------------------------------- Event Tracing -----------------------------------

### **Event Tracing (ETW)**

- **ETW (Event Tracing for Windows):** Windows-wide event collection system with visibility into application and kernel activity.
- **Components:**

| Component   | Purpose                   |
| ----------- | ------------------------- |
| Controllers | Configure/manage sessions |
| Providers   | Generate events           |
| Consumers   | Receive/analyze events    |

- Events use **XML** format and are identified by **Event IDs**.
- **Event ID 4624:** Successful logon.
- ETW provides deeper system visibility than standard event logs; defenders use it for monitoring and detection.

**Link:** [Windows Event Logs – TryHackMe](https://tryhackme.com/room/windowseventlogs?utm_source=chatgpt.com)


### --------------------------------- Approaches to Log Evasion -----------------------------------

### **Approaches to Log Evasion**

- **Log forwarding:** Logs are sent to a centralized server/SIEM, so deleting local logs may not remove forwarded copies.
- **Log tampering risks:** Missing logs or clearing logs can alert defenders and create suspicion (OPSEC risk).
- **Important Event IDs:**

| Event ID | Purpose                             |
| -------- | ----------------------------------- |
| `1102`   | Security audit log cleared          |
| `104`    | Event log cleared                   |
| `1100`   | Windows Event Log service shut down |

\- **Strategic approach:** Understand which logs a technique generates and what defenders monitor. ETW is commonly targeted because it provides control over event tracing.


### --------------------------------- Tracing Instrumentation -----------------------------------

### **Tracing Instrumentation (ETW)**

\- ETW has **3 components** that work together to collect, manage, and analyze events.

| Component       | Purpose                                                   |
| --------------- | --------------------------------------------------------- |
| **Controllers** | Configure tracing sessions and control data flow          |
| **Providers**   | Generate tracing events                                   |
| **Consumers**   | Receive, process, and analyze events (e.g., Event Viewer) |

<img width="881" height="361" alt="image" src="https://github.com/user-attachments/assets/c8bea1d2-6b6c-4794-8bc7-9c06f0a946f7" />

**Provider types:**

- MOF – MOF-based events
- WPP – Uses TMF files to decode traces
- Manifest-Based – Manifest-defined events (up to 8 sessions)
- TraceLogging – Self-describing events (up to 8 sessions)

**Event flow:** `Providers → Controllers/Sessions → Consumers`

**ETW targeting techniques:**

| Component  | Techniques                                                                              |
| ---------- | --------------------------------------------------------------------------------------- |
| Provider   | PSEtwLogProvider Modification, Group Policy Takeover, Log Pipeline Abuse, Type Creation |
| Controller | EtwEventWrite Patching, Runtime Tracing Tampering                                       |
| Consumer   | Log Smashing, Log Tampering                                                             |

**Diagram:** [ETW component interaction](https://cdn-images.tryhackme.com/user-uploads/5e73cca6ec4fcf1309f2df86/room-content/dc8217f5aecbcc08d609c3299756da08.png?utm_source=chatgpt.com)


### --------------------------------- Reflection for Fun and Silence -----------------------------------

### **Reflection for Fun and Silence**

- **Reflection:** .NET technique to inspect and modify assembly metadata, fields, and internal values at runtime.
- **ETW:** PowerShell uses the `PSEtwLogProvider` assembly for event tracing. Reflection can interact with its internal state.
- **Detection impact:** The lesson demonstrates how modifying ETW provider state can reduce PowerShell event visibility (Event ID `4104`), but the modification itself may generate events.
- **Result:** Before modification, commands generated additional events; afterward, command activity was no longer recorded in the same way. The technique itself generated **7 events**, which is an important OPSEC consideration.

**Key terms:** `.NET Assembly`, `Reflection`, `Metadata`, `PSEtwLogProvider`, ETW, Event ID `4104`.


### --------------------------------- Patching Tracing Functions -----------------------------------

### **Patching Tracing Functions**

- **Function patching:** Modifies a function's in-memory instructions so it returns early, preventing its normal execution.
- **ETW:** `EtwEventWrite` is a key function used to write ETW events from user-mode processes.
- **Core concepts:** Disassembly, opcodes, memory permissions, instruction cache, and `ret` (return instruction).
- **Function patching workflow:** Locate function → modify memory permissions → modify instructions → restore permissions → refresh instruction cache.
- **Impact:** Patching ETW can suppress tracing broadly, but may also disrupt legitimate logging and create detectable memory modifications.

**Key terms:** `EtwEventWrite`, `ntdll.dll`, `ret`, Opcode, `VirtualProtect`, `Marshal.Copy`, `FlushInstructionCache`, LIFO.


### --------------------------------- Providers via Policy -----------------------------------

### **Providers via Policy**

- **GPO:** ETW providers can be configured through Group Policy to enable/disable logging features.
- **Script Block Logging:** Records executed PowerShell script blocks.
- **Module Logging:** Records module commands and pipeline activity.

| Event ID | Purpose                              |
| -------- | ------------------------------------ |
| `4103`   | Command invocation / module activity |
| `4104`   | Script block execution and content   |

- **4104:** Provides visibility into executed scripts.
- **4103:** Verbose module logging; generates many events.

**Key point:** PowerShell logging providers can affect visibility, but changing logging configuration may itself be detectable.


### --------------------------------- Group Policy Takeover -----------------------------------

### **Group Policy Takeover**

- **Concept:** PowerShell logging settings are controlled through GPO. Reflection can access cached policy settings at runtime.
- **Affected logging:**
  - `EnableScriptBlockLogging` → Event ID `4104`
  - `EnableScriptBlockInvocationLogging` → Event ID `4103`
- **Impact:** Modifying provider settings can reduce PowerShell logging visibility without clearing event logs.
- **Detection:** Monitor changes to PowerShell logging policies, suspicious reflection activity, and unexpected gaps in Event IDs `4103`/`4104`.


### --------------------------------- Abusing Log Pipeline -----------------------------------

### **Abusing Log Pipeline**

- **Concept:** PowerShell modules and snap-ins have `LogPipelineExecutionDetails`, which controls pipeline execution logging.
- **Impact:** Changing logging settings can reduce module/cmdlet activity visibility for the current session.
- **Scope:** Applies to imported modules and snap-ins; it does not automatically disable all Windows logging.
- **Detection:** Monitor unexpected changes to PowerShell logging configuration and gaps in module activity logs.

**Key term:** `LogPipelineExecutionDetails` — controls module pipeline logging.




# Living Off the Land

### --------------------------------- Windows Sysinternals -----------------------------------

### **Windows Sysinternals**

- **Sysinternals Suite:** Collection of Windows utilities for system administration, troubleshooting, monitoring, and diagnostics.
- **Categories:** Disk, process, networking, system information, security.
- **License:** Accept using `-accepteula`.

| Tool                                                                                                    | Purpose                                              |
| ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| [AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk?utm_source=chatgpt.com)   | Check permissions on files, Registry, services, etc. |
| [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec?utm_source=chatgpt.com)         | Execute programs on remote systems                   |
| [ADExplorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer?utm_source=chatgpt.com) | Explore/manage Active Directory                      |
| [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump?utm_source=chatgpt.com)     | Monitor processes & dump memory                      |
| [ProcMon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon?utm_source=chatgpt.com)       | Monitor process activity                             |
| [TCPView](https://docs.microsoft.com/en-us/sysinternals/downloads/tcpview?utm_source=chatgpt.com)       | View TCP/UDP connections                             |
| [PsTools](https://docs.microsoft.com/en-us/sysinternals/downloads/pstools?utm_source=chatgpt.com)       | Collection of system information/utilities           |
| [Portmon](https://docs.microsoft.com/en-us/sysinternals/downloads/portmon?utm_source=chatgpt.com)       | Monitor serial/parallel ports                        |
| [Whois](https://docs.microsoft.com/en-us/sysinternals/downloads/whois?utm_source=chatgpt.com)           | Domain/IP information                                |

**Sysinternals Live:** Tools can be used without installation via browser, network share, or command line.

- Live: [https://live.sysinternals.com/](https://live.sysinternals.com/?utm_source=chatgpt.com)
- Network share: `\\live.sysinternals.com\tools`
- Local tools: `C:\Tools\SysinternalsSuite\`

```
C:\Tools\SysinternalsSuite\PsExec64.exe
```

**Red Team relevance:** Trusted Microsoft utilities can be abused by attackers (Living-off-the-Land), but defenders also monitor their suspicious usage.

**Resources:**

- [TryHackMe Sysinternals](https://tryhackme.com/room/btsysinternalssg?utm_source=chatgpt.com)
- [Microsoft Sysinternals Resources](https://docs.microsoft.com/en-us/sysinternals/resources/?utm_source=chatgpt.com)


### --------------------------------- LOLBAS Project -----------------------------------

### **LOLBAS Project**

- **LOLBAS:** *Living Off The Land Binaries And Scripts* — a community repository of Microsoft-signed, built-in binaries, scripts, and libraries that can be abused for unintended functionality.
- **Purpose:** Search tools by binary, function, type, and MITRE ATT&CK information.
- **Types:** Script, Binary, Libraries, OtherMSBinaries.
- **Search syntax:**
  - `binary-name` → search for a binary
  - `/execute` → search by function
  - `#type` → search by type

**Accepted functionality:** Code execution, file operations, compilation, persistence, UAC bypass, memory dumping, DLL injection, and more.

**Criteria:** Microsoft-signed/native or Microsoft-downloaded, has unintended functionality, and is useful for Red Team/APT operations.

**Link:** [LOLBAS Project](https://lolbas-project.github.io/?utm_source=chatgpt.com) | [GitHub](https://github.com/LOLBAS-Project/LOLBAS?utm_source=chatgpt.com)


### --------------------------------- File Operations -----------------------------------

### **File Operations (LOLBAS)**

| Tool          | Purpose                                                 | ATT&CK       |
| ------------- | ------------------------------------------------------- | ------------ |
| **CertUtil**  | Certificate management; also file transfer and encoding | T1105, T1027 |
| **BITSAdmin** | Manage asynchronous BITS download/upload jobs           | T1197        |
| **FindStr**   | Search text/patterns in files and command output        | —            |

- **CertUtil:** File transfer and encoding/decoding; originally intended for certificate management.
- **BITSAdmin:** Uses BITS for low-bandwidth, asynchronous file transfers over HTTP/SMB.
- **FindStr:** Searches strings in files/output; can access files from SMB shares.

**References:**

- [CertUtil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil?utm_source=chatgpt.com)
- [BITS](https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal?utm_source=chatgpt.com)
- [BITSAdmin](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/bitsadmin?utm_source=chatgpt.com)
- [FindStr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr?utm_source=chatgpt.com)
- [LOLBAS](https://lolbas-project.github.io/?utm_source=chatgpt.com)


### --------------------------------- File Execution -----------------------------------

### **File Execution (LOLBAS)**

- **Indirect Command Execution:** Executes programs through trusted Windows binaries instead of launching them directly.
- **Signed Binary Proxy Execution (T1218):** Abuses legitimate, signed Windows utilities to execute malicious code, potentially bypassing security controls.

| Tool           | Purpose                                   | Technique          |
| -------------- | ----------------------------------------- | ------------------ |
| `explorer.exe` | Launches files through File Explorer      | Indirect execution |
| `wmic.exe`     | Creates/manages processes through WMI     | T1218              |
| `rundll32.exe` | Loads DLLs and can proxy script execution | T1218.011          |

- **Why attackers use it:** Trusted system binaries may make malicious activity harder to distinguish from legitimate activity.
- **Defensive focus:** Monitor unusual parent-child processes, suspicious command-line arguments, unexpected DLL loading, and misuse of signed Windows binaries.

**Reference:** [MITRE ATT&CK – Rundll32 (T1218.011)](https://attack.mitre.org/techniques/T1218/011/?utm_source=chatgpt.com)


### --------------------------------- Application Whitelisting Bypasses -----------------------------------

### **Application Whitelisting Bypasses**

- **Application Whitelisting:** Allows only approved applications to execute; blocks unauthorized programs.
- **LOLBAS:** Trusted Microsoft binaries can be abused to execute unauthorized code.
- `regsvr32.exe`**&#x20;(T1218.010):** DLL registration tool that can also proxy execution of malicious code. Located in `System32` and `SysWOW64`.
- `bash.exe`**&#x20;(T1202):** Windows Subsystem for Linux (WSL) binary; can be abused for indirect command execution.
- **Detection:** Monitor unusual usage of trusted binaries, suspicious command-line arguments, network activity and unexpected child processes.

| Binary         | Legitimate purpose             | Abuse              |
| -------------- | ------------------------------ | ------------------ |
| `regsvr32.exe` | Register/unregister DLLs       | Proxy execution    |
| `bash.exe`     | Run Linux commands through WSL | Indirect execution |

**References:**

- [LOLBAS](https://lolbas-project.github.io/?utm_source=chatgpt.com)
- [MITRE T1218](https://attack.mitre.org/techniques/T1218/?utm_source=chatgpt.com)
- [MITRE T1202](https://attack.mitre.org/techniques/T1202/?utm_source=chatgpt.com)
- [WSL Documentation](https://docs.microsoft.com/en-us/windows/wsl/about?utm_source=chatgpt.com)


### --------------------------------- Other Techniques -----------------------------------

\- **Shortcut Modification (T1547.009):** Modified `.lnk` shortcuts can execute a different program instead of the intended application, potentially enabling initial access or persistence.

- Target can reference: `rundll32.exe`, PowerShell, `regsvr32.exe`, or another executable.
- Example: A fake Excel shortcut launches another program.
- [Atomic Red Team](https://github.com/theonlykernel/atomic-red-team/blob/master/atomics/T1023/T1023.md?utm_source=chatgpt.com)

\- **PowerLessShell:** Uses **MSBuild** to execute PowerShell code without spawning a visible `powershell.exe` process.

- **MSBuild:** Microsoft Build Engine used to build .NET applications.
- Useful concept: PowerShell execution can occur without the usual PowerShell process being visible.
- **Detection:** Monitor unusual `MSBuild.exe` execution, suspicious `.csproj` files, unexpected child processes and network activity.



