# Chapter 2: WinDbg and x86 Architecture
## Chapter 2.1.1. Program Memory
### Stack
<img width="400" height="250" alt="image" src="https://github.com/user-attachments/assets/9212dbe5-1459-4ae2-be34-25bdbcab8dd0" /> 

- **Thread**: Running code from Program Image or from Dynamic Link Libraries (DLLs)
- **Stack(LIFO)**: contain temporary data area for functions, local variable and program control information when thread is running

---

### Calling conventions
[Calling conventions](https://en.wikipedia.org/wiki/Calling_convention) describe **how functions receive parameters from their caller** and **how they return results**.  
They form part of the **Application Binary Interface (ABI)** and act as a contract between caller and callee.

Calling conventions differ based on:

- **Parameter passing**  
  - In CPU registers, on the stack, or a combination
- **Parameter order**  
  - Left-to-right, right-to-left, or defined by convention
- **Return value handling**  
  - Via registers, stack, or memory references
- **Register preservation**  
  - Some registers must retain caller values; others are considered volatile
- **Stack management**  
  - Which side handles stack setup and cleanup after a function call
- **Variadic function handling**  
  - Passing variable numbers of parameters
- **Frame pointer management**  
  - Storage and restoration of previous frame state
- **Object/reference passing**  
  - Special considerations in object-oriented languages

---

### Function Return Mechanics
<img width="250" height="100" alt="image" src="https://github.com/user-attachments/assets/38c35eae-fe05-4b06-ba26-58cce3c86bf9" />

When a function is called:

1. **Return address storage**  
   - The CPU saves the caller’s return address (usually on the stack).
2. **Stack frame creation**  
   - A region of stack memory is allocated for:
     - Return address  
     - Function parameters  
     - Local variables  
3. **Execution of the callee**  
   - The called function runs with its local stack frame.
4. **Return sequence**  
   - The CPU restores the return address from the stack back into the instruction pointer.  
   - Execution resumes in the calling function.

---

### CPU Registers
CPU registers are **tiny, ultra-fast storage areas** inside the processor.  
On a 32-bit x86 CPU, there are nine important 32-bit registers used to make programs run efficiently.

---

## What Is a Register?

- A *register* stores small amounts of data that the CPU can read and change very quickly.  
- Registers work much faster than regular memory (RAM).

---

## General Purpose Registers

These are used for all sorts of operations – math, memory address calculations, or temporarily holding data.  
Each register has a name and a typical use:

| Register | Name (What it means) | Typical Use |
|----------|---------------------|-------------|
| **EAX**  | Accumulator         | Arithmetic and logic (main math work) |
| **EBX**  | Base                | Base address for memory access        |
| **ECX**  | Counter             | Loop and rotation counters            |
| **EDX**  | Data                | I/O, extra multiplication/division    |
| **ESI**  | Source Index        | Source address for string operations  |
| **EDI**  | Destination Index   | Destination for string operations     |

---

## ESP – The Stack Pointer

- **ESP** keeps track of the *top of the stack* in memory.
- The stack is used to store data like function calls, return addresses, and local variables.
- When something is pushed to the stack, ESP gets smaller; when it’s popped, ESP grows.

> **A pointer references a specific memory address. ESP always shows the address for the top of the stack—where the next item will be placed or removed.**

---

## EBP – The Base Pointer

- **EBP** helps functions find their place (stack frame) in the constantly changing stack.
- When a function starts:
  - The previous EBP is saved.
  - EBP is set to ESP’s current value, marking the *start of the stack frame*.
  - ESP then allocates the space for the function’s local variables.

---

## Subregisters: 16- and 8-bit Parts

- Register names like **EAX**, **EBX**, etc., are for 32 bits.
- They also have 16- or 8-bit versions (like AX, AH, AL) for working with less data when needed.

---

