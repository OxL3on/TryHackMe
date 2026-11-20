## Windows Internals

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


### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------
### --------------------------------- Processes -----------------------------------

