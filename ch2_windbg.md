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

- **db**: Display bytes (8 bits)
  - Example: `db esp`
  - Shows content byte-by-byte at the address in the ESP register
- **dw**: Display words (16 bits, 2 bytes)
  - Example: `dw esp`
  - Shows two bytes at a time
- **dd**: Display double-words (32 bits, 4 bytes)
  - Example: `dd esp`
  - Shows four bytes at a time
- **dq**: Display quad-words (64 bits, 8 bytes)
  - Example: `dq esp`
  - Shows eight bytes at a time

### Displaying by Format and Symbol

- You can use registers (like `esp`), explicit memory addresses, or symbol names (like `kernel32!WriteFile`) with these commands.

### Displaying Characters and ASCII

- **dc**: Display DWORDs and ASCII characters together
- **dW**: Display words and ASCII (don’t confuse with `dw`)
- ASCII characters appear to the right in these formats for easy reading.

### Specifying Range and Length

- The `L` parameter sets the amount of data to display.
  - Example: `dd esp L4` (display four DWORDs)

### Composite and Pointer Commands

- **poi**: Dereferences a pointer—useful for following pointers in memory
  - Example: `dd poi(esp)`

Shows the same memory as a byte, word, and dword.

### Special Formats

- **da**: Displays ASCII strings
- **du**: Displays Unicode strings
- **dW**: Shows words and ASCII
- **dc**: Shows DWORDs and their corresponding ASCII

**Tip:**  
- Use the appropriate command based on the data format you need to inspect.
- The `L` parameter helps control how much data is displayed at once.
- `poi()` lets you read the value pointed to by a pointer (dereference).

These commands are essential for investigating the raw data held in process memory when debugging or reverse engineering applications.
