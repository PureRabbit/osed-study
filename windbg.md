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

### Unassembling Memory

`u` is used to inspect assembly code of certain windows APIs
- The `u` command displays the assembly translation for a specific memory address or a range of memory.
- If no address is given, it starts from the address stored in the EIP register (the current instruction).

### Typical Workflow Example

1. **Start WinDbg** from the Taskbar and open your target application, e.g., Notepad.
2. **Attach the debugger** to the process (e.g., `notepad.exe`).
3. If the application is running, use the "Break" option in the Debug menu to pause execution.
4. Once suspended, use the `u` command to display the assembly of a specific function.

**Important Notes:** Pausing (breaking) ensures the debugger can read from memory without interference from the program changing that memory, which allows for reliable disassembly and inspection.


`u kernel32!GetCurrentThread` to disassemble the target function

| Instruction        | Meaning                                    | Effect                     |
|--------------------|--------------------------------------------|----------------------------|
| push 0xFFFFFFFE    | Push -2 onto the stack                     | Stack top = -2             |
| pop eax            | Move value (-2) from stack to eax register | eax = -2                   |
| ret                | Return from function                       | Caller gets eax as result  |
| int 3              | Debugging breakpoint (not part of result)  | Used for debugger break    |
