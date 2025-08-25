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

---

✅ **Tip for Learners:**  
Always load valid **symbol files** in WinDbg. Without symbols, the `dt` command cannot display meaningful names or layouts.  











