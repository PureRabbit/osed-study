# Chapter 2: WinDbg and x86 Architecture
## Chapter 2.2 Introduction to Windows Debugger

### Debugging Symbols

Symbol files (`.PDB`) allow WinDbg to identify functions, structures, and variables by name instead of memory addresses. 

These files are generated during compilation and can be retrieved from the Microsoft Symbol Store or from third-party application providers.

The symbol path can be set via **File > Symbol File Path...**, with a common location being `C:\symbols`.

To load symbols before debugging, the `.reload` command is used. Errors may occur if symbols are unavailable, particularly after Windows updates or when no symbol file exists for a module. Troubleshooting typically involves:
- Enabling diagnostics with `!sym noisy`
- Verifying the symbol search path with `.sympath`

Most critical module symbols become available within days of Windows patches, so errors usually do not prevent debugging.

## Chapter 2.3.1 Unassembling Memory

`u` is used to inspect assembly code of certain windows APIs
- The `u` command displays the assembly translation for a specific memory address or a range of memory.
- If no address is given, it starts from the address stored in the EIP register (the current instruction).

### Typical Workflow Example

1. **Start WinDbg** from the Taskbar and open your target application, e.g., Notepad.
2. **Attach the debugger** to the process (e.g., `notepad.exe`).
3. If the application is running, use the `"Debug > Break"` option in the Debug menu to pause execution. But normally its paused. `g` makes the program run.
4. Once suspended, use the `u` command to display the assembly of a specific function.

**Important Notes:** Pausing (breaking) ensures the debugger can read from memory without interference from the program changing that memory, which allows for reliable disassembly and inspection.

`u kernel32!GetCurrentThread` to disassemble the target function

| Instruction        | Meaning                                    | Effect                     |
|--------------------|--------------------------------------------|----------------------------|
| push 0xFFFFFFFE    | Push -2 onto the stack                     | Stack top = -2             |
| pop eax            | Move value (-2) from stack to eax register | eax = -2                   |
| ret                | Return from function                       | Caller gets eax as result  |
| int 3              | Debugging breakpoint (not part of result)  | Used for debugger break    |

---

## 2.3.2. Reading from Memory with WinDbg

WinDbg provides several commands to **display and inspect memory content** of a process. These commands help you view memory in different data sizes and formats.

### Key Display Commands

| Command | Usage Description                                                         | Example |
|---------|--------------------------------------------------------------------------|---------|
| db      | Display bytes and ASCII characters at the specified address              |   <img width="554" height="180" alt="image" src="https://github.com/user-attachments/assets/caabb8e6-f14e-4730-88ef-b3138ead2779" /> |
| dw      | Display word (2 bytes) values                                            |   <img width="374" height="181" alt="image" src="https://github.com/user-attachments/assets/a1763158-9dfd-45a9-918d-a2db6cd7f746" /> |
| dd      | Display double-word (4 bytes) values                                     |   <img width="340" height="170" alt="image" src="https://github.com/user-attachments/assets/be896f08-9260-4afb-b285-ffd3a447f193" /> |
| dq      | Display quad-word (8 bytes) values                                       |   <img width="335" height="176" alt="image" src="https://github.com/user-attachments/assets/1f779514-fbb7-4366-a3a2-6f44d98edbc3" /> |
| da      | Display memory as ASCII string                                           |         |
| du      | Display memory as Unicode string                                         |         |
| dc      | Display double-word values and corresponding ASCII characters            |         |
| dW      | Display word values and ASCII characters (not to be confused with dw)    |    <img width="485" height="176" alt="image" src="https://github.com/user-attachments/assets/9996f5f1-8c18-42a7-88fd-e5aa3a36821b" /> |
| df      | Display single-precision floating-point numbers                          |         |
| dD      | Display double-precision floating-point numbers                          |         |
| dp      | Display pointer-sized values (like dd or dq, depending on architecture)  |         |
| dyb     | Display binary and byte values                                           |         |
| dyd     | Display binary and double-word values                                    |         |
| poi     | Dereference a pointer from a memory address                             |   <img width="342" height="181" alt="image" src="https://github.com/user-attachments/assets/b558a2cd-b5ff-42de-9844-2afe3fbd36b1" />  |

### Displaying by Format and Symbol

- You can use registers (like `esp`), explicit memory addresses, or symbol names (like `kernel32!WriteFile`) with these commands.

**Tip:**  
- Use the appropriate command based on the data format you need to inspect.
- The `L` parameter helps control how much data is displayed at once.
- `poi()` lets you read the value pointed to by a pointer (dereference).

These commands are essential for investigating the raw data held in process memory when debugging or reverse engineering applications.

### Reversing Order
The order appears reversed in the output of the db and dw commands because of endianness. Most Windows systems use little-endian byte order, where the least significant byte is stored at the lowest memory address.
- db (display bytes) shows memory byte-by-byte, so they are displayed in increasing memory address order.
- dw (display words) shows 2-byte "word" values, but each word is shown in host (little-endian) order—so the lower byte (stored first in memory) is shown last within each 2-byte group.

**Example for address 1000 containing 0x12 0x34:**
- `db 1000 L2 → 12 34`    (memory order: low to high)
- `dw 1000 L1 → 3412`    (host byte order: least significant byte appears last)

  
Little-endian means the least significant part comes first in memory, causing the "reversal" when grouping bytes into words or larger data types for display.


<img width="489" height="49" alt="image" src="https://github.com/user-attachments/assets/052b3aa9-904e-4338-be29-2c93f073e450" />


The hexadecimal value 5A4D corresponds to the ASCII characters 'Z' and 'M'.
`0x5A` = 'Z' (Higher memory)
`0x4D` = 'M' (Lower memory)
So, 5A4D in ASCII is "ZM".

## Chapter 2.3.3 Dumping Structure from Memory

### Key Concept
- **Structures**: Collections of fields (ints, pointers, or nested structures) defined in program code.  
- **Problem**: After code is compiled, structures turn into binary data, making them unreadable.  
- **Solution in WinDbg**: Use **symbols** and the `dt` (Display Type) command to **reconstruct and dump structures** from memory.  
- **Benefits**:  
  - Human-readable names instead of raw memory offsets.  
  - Ability to explore nested structures.  
  - Useful in debugging as Windows APIs often use structures.  

### Command Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `dt Module!Structure` | Displays structure definition (fields, offsets, types) from symbols | `dt ntdll!_TEB` |
| `dt Module!Structure Address` | Dumps the structure at a **specific memory address** | `dt ntdll!_TEB @$teb` |
| `dt -r Module!Structure Address` | **Recursive dump**: expands nested structures automatically | `dt -r ntdll!_TEB @$teb` |
| `dt Module!Structure Address FieldName` | Displays only a **specific field** of a structure | `dt ntdll!_TEB @$teb ThreadLocalStoragePointer` |
| `?? sizeof(Module!Structure)` | Shows the **size of a structure** in bytes | `?? sizeof(ntdll!_TEB)` |


✅ **Tip for Learners:**  
- Always load valid **symbol files** in WinDbg. Without symbols, the `dt` command cannot display meaningful names or layouts.  
- ntdll (ntdll.dll) is a core Windows system library that provides the Windows Native API, containing low-level NT kernel functions essential for the operating system’s core functionality. Most applications do not call it directly; instead, it supports system-level operations required by the OS and other system libraries.
- 1 thread only 1 TEB. Use NTDLL to access the TEB related information.
- `@$teb` is a pseudo-register in WinDbg that represents the **memory address of the current thread’s TEB (Thread Environment Block)**
- 1 process has 1 Process Environment Block (PEB) shared by all threads in that process
- the _TEB structure is defined by the Windows operating system and is always associated with system modules, typically ntdll!_TEB or nt!_TEB. Other application or third-party modules do not define their own versions of the TEB—only the official Windows headers and core modules include it. Attempting to use a different module’s _TEB (such as othermodule!_TEB) would not work, because only the system-provided ones match the actual layout and address used by the OS.
- Every thread in a Windows application does have a TEB—but it is created and managed by the Windows operating system, not by each application itself. 

```
TEB (Thread Environment Block)
│
├─ NtTib : _NT_TIB
│ ├─ ExceptionList : _EXCEPTION_REGISTRATION_RECORD
│ │ ├─ Next : _EXCEPTION_REGISTRATION_RECORD
│ │ └─ Handler : _EXCEPTION_DISPOSITION
│ ├─ StackBase : Pointer (stack upper limit)
│ ├─ StackLimit : Pointer (stack lower limit)
│ ├─ SubSystemTib : Pointer
│ ├─ FiberData / Version
│ ├─ ArbitraryUserPointer
│ └─ Self : Pointer to _NT_TIB (recursive link)
│
├─ EnvironmentPointer : Pointer
├─ ClientId : _CLIENT_ID
│ ├─ UniqueProcess : Process ID
│ └─ UniqueThread : Thread ID
│
├─ ActiveRpcHandle : Pointer
├─ ThreadLocalStoragePointer : Pointer
├─ ProcessEnvironmentBlock : _PEB
│ ├─ InheritedAddressSpace
│ ├─ BeingDebugged
│ ├─ ImageBaseAddress
│ ├─ Ldr : _PEB_LDR_DATA
│ └─ ... (further system & process information)
│
├─ LastErrorValue : Integer
├─ CountOfOwnedCriticalSections : Integer
└─ ... (other thread-related fields up to offset 0x1000)
```

In Windows, each TEB (Thread Environment Block) contains a pointer to the PEB (Process Environment Block) because the TEB is thread-specific while the PEB is process-wide. This design allows each thread to quickly access both its own thread-local information (via the TEB) and shared process data (via the PEB) without complex lookups


## Chapter 2.3.4 ~ 2.3.6.: Writing, Searching, and Editing Memory & Registers

### Key Concepts

- **Writing to Memory:**  
  - WinDbg lets you directly modify memory in a process using the `edit` commands.  
  - You can write integers, ASCII, or Unicode strings to any address.

- **Searching Memory:**  
  - The `search` command (`s`) helps locate values, strings, or patterns within a specified memory range.  
  - Useful for reverse engineering and exploit development.

- **Inspecting/Editing CPU Registers:**  
  - The `r` command allows you to view and change CPU register values.  
  - This is vital for low-level debugging and crafting exploits.

---

### Command Summary Table

| Command                        | Purpose                                                            | Example                                                        |
|---------------------------------|--------------------------------------------------------------------|----------------------------------------------------------------|
| `ed Address Value`              | Edit (write) a DWORD value at Address                              | `ed esp 41414141`                                              |
| `ea Address "String"`           | Edit (write) an ASCII string at Address                            | `ea esp "Hello"`                                               |
| `eu Address "String"`           | Edit (write) a Unicode string at Address                           | `eu esp "Test"`                                                |
| `s -d Start Length Pattern`     | Search for a **DWORD** value in memory                             | `s -d 0 L?80000000 41414141`                                   |
| `s -a Start Length "String"`    | Search for an **ASCII string** in memory                           | `s -a 0 L?80000000 "This program cannot be run in DOS mode"`   |
| `r`                             | Display all CPU register values                                    | `r`                                                            |
| `r reg`                         | Display a single register's value                                  | `r ecx`                                                        |
| `r reg=Value`                   | Set a register to a new value                                      | `r ecx=41414141`                                               |

**Tip for Learners:**  
Practice modifying different memory locations and register values in a test process. Always understand the effect of each change before applying it, especially with live or critical software.

---

## Chapter 2.4.1 ~ 2.4.3.: WinDbg Program Execution Control – Summary

**WinDbg** uses breakpoints and step commands to control program execution for debugging purposes. There are two primary breakpoint types: **software (bp)**, which temporarily modifies program code, and **hardware (ba)**, which uses CPU debug registers to monitor memory and execution. Stepping commands allow execution to proceed instruction-by-instruction, either diving into or stepping over function calls. Breakpoints can be automated and made conditional, making WinDbg a powerful tool for code analysis.

### Key Concepts

### Breakpoints
- **Software Breakpoints (`bp`)**: Set by the debugger at specific locations; unlimited in number; modifies code at runtime.
- **Hardware Breakpoints (`ba`)**: Set by CPU, monitor read/write/execute access at memory addresses; limited by CPU debug registers; does not modify code.
- **Unresolved Breakpoints (`bu`)**: For functions not yet loaded; become active when the module loads.
- **Breakpoint Management**: Commands exist to list, enable, disable, or clear breakpoints.

### Stepping Through Code
- **Step Over (`p`)**: Executes the next instruction but skips over function calls.
- **Step Into (`t`)**: Executes the next instruction and dives into function calls.
- **Step Out (`gu`)**: Runs until current function returns.
- **Continue (`g`)**: Runs until the next breakpoint or exception.
- **Step to Return (`pt`)**: Executes until the current function returns.
- **Step to Branch (`ph`)**: Executes until a branch instruction (call/ret/jmp).

### Conditional and Automated Breakpoints
- **Breakpoint Actions**: Automate logger or actions when a breakpoint is hit, e.g. print data, run scripts.
- **Conditional Breakpoints**: Halt only when certain conditions are true using `.if` syntax in breakpoint command[web:6].

---

### WinDbg Program Control Commands Table

| Command                  | Description                                               | Example Use                                |
|--------------------------|----------------------------------------------------------|--------------------------------------------|
| `bp [location]`          | Set a software breakpoint                                | `bp kernel32!WriteFile`                    |
| `ba [type] [size] [addr]`| Set a hardware/data breakpoint (e=exec, r=read, w=write) | `ba w 4 0x12345678`                        |
| `bu [symbol]`            | Set unresolved breakpoint (module not loaded)            | `bu ole32!WriteStringStream`               |
| `bl`                     | List all breakpoints and their status                    | `bl`                                       |
| `be [num]`               | Enable breakpoint by number                              | `be 0`                                     |
| `bd [num]`               | Disable breakpoint by number                             | `bd 0`                                     |
| `bc [num|*] `            | Clear a breakpoint by number or all (`*`)                | `bc 1` / `bc *`                            |
| `lm`                     | list all modules / `lm m [module]`                       | `lm / lm m ole32 `                         |
| `g`                      | Resume execution                                         | `g`                                        |
| `p`                      | Step over (next instruction, skip calls)                 | `p`                                        |
| `t`                      | Step into (next instruction, enter calls)                | `t`                                        |
| `pt`                     | Step to next return                                      | `pt`                                       |
| `ph`                     | Step to next branch (call/ret/jmp)                       | `ph`                                       |
| `gu`                     | Step out (run until function returns)                    | `gu`                                       |
| `.if/.else`              | Use conditionals for breakpoint actions                  | `bp ... ".if (condition) {g} .else {break};"` |
| `[command] [{args}] ; ...` | Automate command(s) when breakpoint hits                | `bp ... ".printf \"Value: %p\", @eax; g"`  |

*This table uses real WinDbg command syntax as referenced from community and official Quick Reference sources.*[web:6][web:9]

### Additional 
- **ole32.dll** is a core Windows system library responsible for OLE (Object Linking and Embedding) functions, which allow applications to embed and link to documents and objects across different programs. This enables features like embedding an Excel spreadsheet in a Word document or inserting images/objects into files. The library manages object creation, data transfer, and inter-process communication in the OLE environment, supporting software interoperability on Windows.
- for the `ba w 2 03b2c768` command, the number represent the number of byte to watch. It usaully in 1/2/4/8. if you set `ba w 2 03b2c768`, the program will break (pause execution) when any write operation modifies any of the 2 bytes at the memory address 0x03b2c768.



| Feature                  | Hardware Breakpoint                              | Software Breakpoint                                   |
|--------------------------|--------------------------------------------------|-------------------------------------------------------|
| How it works             | Uses special CPU debug registers to monitor specific addresses or instructions | Modifies program code in memory with a special breakpoint instruction (e.g., INT 3 on x86) |
| Quantity Limit           | Limited (usually 2-4 per CPU/thread on x86/x64)  | Essentially unlimited (restricted only by memory size)|
| Location restriction     | Works in any memory area, including read-only/flash | Only works in writable memory (usually RAM), not in ROM/flash |
| Can break on data access | Yes (read, write, or execute access)             | Normally only on instruction execution                |
| Performance              | Very fast, negligible overhead                   | Slightly slower, incurs overhead due to code patching |
| Typical use cases        | Tracking memory reads/writes, code in flash, rare bug conditions | General debugging, multiple breakpoints on code       |
| Drawbacks                | Very limited in number; may require special permissions | Cannot be used on code in ROM or flash; risks leaving patched code if broken debugger session |
| Example WinDbg command   | `ba w 4 0x12345678`                              | `bp mymodule!myfunction`                              |



## Chapter 2.4.4 :Key Concept: Stepping Through Code in WinDbg

Stepping through code is essential in reverse engineering and debugging to see exactly how a program executes, line by line or instruction by instruction. WinDbg provides commands that let you control program flow at this granular level, allowing you to skip over or dive into functions to understand their actual behavior.

---

## Command Summary Table

| Command   | Name/Action             | What It Does                                                                                 | When to Use                             |
|-----------|-------------------------|---------------------------------------------------------------------------------------------|-----------------------------------------|
| `p`       | Step Over               | Executes the next instruction; if it’s a function call, runs the whole function and stops after it returns | When you want to skip function details  |
| `t`       | Step Into               | Executes the next instruction; if it’s a function call, goes into the function, stopping at its first instruction | When you want to see inside the function|
| `pt`      | Step to Return          | Executes instructions until a `ret` (return) instruction is hit in the current function      | To quickly jump out of the current function |
| `ph`      | Step to Branch          | Runs until any branch (call, jump, return, etc.) instruction is reached                      | To fast-forward to the next branch      |
| `g`       | Go (Continue)           | Runs the program until the next breakpoint or exception                                      | To resume normal execution              |
| `gu`      | Step Out                | Runs until current function returns                                                          | When you’re done examining a function   |

---



