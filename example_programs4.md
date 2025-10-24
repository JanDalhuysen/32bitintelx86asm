# System Call Examples in 32-bit Intel x86 Assembly

This document contains examples of system calls in 32-bit Intel x86 assembly using NASM syntax.

## Table of Contents

1. [Writing to Standard Output](#writing-to-standard-output)
2. [Reading from Standard Input](#reading-from-standard-input)
3. [File Operations](#file-operations)
4. [Process Exit](#process-exit)

---

## Writing to Standard Output

### Hello World with System Call

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Simple "Hello, World!" program using system calls
;*****************************************************************************
section .data
  msg db 'Hello, World!', 10  ; Message string with newline (10 = \n)
  msg_len equ $ - msg         ; Calculate length of message

section .text
  global _start

_start:
  ; System call: sys_write
  mov   eax, 4        ; sys_write system call number (4)
  mov   ebx, 1        ; file descriptor 1 = stdout
  mov   ecx, msg      ; pointer to message string
  mov   edx, msg_len  ; length of message
  int   0x80          ; invoke system call

  ; Exit program
  mov   eax, 1        ; sys_exit system call number (1)
  mov   ebx, 0        ; exit status 0 (success)
  int   0x80          ; invoke system call
```

### Writing Custom String

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Write a custom string to stdout using system calls
;*****************************************************************************
section .data
  greeting db 'Welcome to Assembly Programming!', 10
  greeting_len equ $ - greeting

section .text
  global _start

_start:
  ; System call: write to stdout
  mov   eax, 4              ; sys_write = 4
  mov   ebx, 1              ; stdout = 1
  mov   ecx, greeting       ; address of string
  mov   edx, greeting_len   ; length of string
  int   0x80                ; make system call

  ; Exit gracefully
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 0              ; exit code 0 (success)
  int   0x80                ; make system call
```

---

## Reading from Standard Input

### Reading User Input

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Read input from stdin using system calls
;*****************************************************************************
section .bss
  buffer resb 256           ; Reserve 256 bytes for input buffer

section .text
  global _start

_start:
  ; System call: read from stdin
  mov   eax, 3              ; sys_read = 3
  mov   ebx, 0              ; stdin = 0
  mov   ecx, buffer         ; buffer address
  mov   edx, 256            ; max bytes to read
  int   0x80                ; make system call

  ; The number of bytes actually read is returned in eax
  ; Store it in a register for later use
  mov   ebx, eax            ; save number of bytes read

  ; Now write the input back to stdout
  mov   eax, 4              ; sys_write = 4
  mov   ecx, buffer         ; buffer address
  mov   edx, ebx            ; number of bytes to write
  int   0x80                ; make system call

  ; Exit program
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 0              ; exit code 0
  int   0x80                ; make system call
```

---

## File Operations

### Creating and Writing to a File

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Create a file and write data to it using system calls
;*****************************************************************************
section .data
  filename db 'output.txt', 0    ; Null-terminated filename
  data db 'Hello from assembly!', 10  ; Data to write
  data_len equ $ - data

section .text
  global _start

_start:
  ; System call: create file
  mov   eax, 8              ; sys_creat = 8
  mov   ebx, filename       ; filename address
  mov   ecx, 0644o          ; file permissions (read/write for owner, read for group/other)
  int   0x80                ; make system call
  ; File descriptor returned in eax

  ; Save file descriptor for later use
  mov   ebx, eax            ; ebx now contains file descriptor

  ; System call: write to file
  mov   eax, 4              ; sys_write = 4
  mov   ecx, data           ; data to write
  mov   edx, data_len       ; length of data
  int   0x80                ; make system call

  ; System call: close file
  mov   eax, 6              ; sys_close = 6
  mov   ebx, eax            ; file descriptor to close
  int   0x80                ; make system call

  ; Exit program
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 0              ; exit code 0
  int   0x80                ; make system call
```

### Opening and Reading a File

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Open and read a file using system calls
;*****************************************************************************
section .data
  filename db 'input.txt', 0     ; Null-terminated filename

section .bss
  file_buffer resb 1024          ; Buffer to store file contents

section .text
  global _start

_start:
  ; System call: open file
  mov   eax, 5              ; sys_open = 5
  mov   ebx, filename       ; filename address
  mov   ecx, 0              ; flags: O_RDONLY
  int   0x80                ; make system call
  ; File descriptor returned in eax

  ; Save file descriptor
  mov   ebx, eax            ; ebx now contains file descriptor

  ; System call: read from file
  mov   eax, 3              ; sys_read = 3
  mov   ecx, file_buffer    ; buffer address
  mov   edx, 1024           ; max bytes to read
  int   0x80                ; make system call
  ; Number of bytes read returned in eax

  ; Save number of bytes read
  mov   edx, eax            ; edx now contains bytes read

  ; System call: write to stdout (to display file contents)
  mov   eax, 4              ; sys_write = 4
  mov   ebx, 1              ; stdout = 1
  mov   ecx, file_buffer    ; buffer address
  ; edx already contains number of bytes to write
  int   0x80                ; make system call

  ; System call: close file
  mov   eax, 6              ; sys_close = 6
  mov   ebx, [file_descriptor]  ; file descriptor to close (would need to store it first)
  int   0x80                ; make system call

  ; For this example, we'll just exit since we didn't store the fd
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 0              ; exit code 0
  int   0x80                ; make system call

section .data
  file_descriptor dd 0      ; Storage for file descriptor (added for completeness)
```

---

## Process Exit

### Different Exit Codes

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* Program with different exit codes based on conditions
;*****************************************************************************
section .text
  global _start

_start:
  ; Simulate some computation that determines exit status
  mov   eax, 42             ; Some result value
  cmp   eax, 42             ; Compare with expected value
  je    .success            ; Jump if equal

  ; Failure case
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 1              ; exit code 1 (failure)
  int   0x80                ; make system call

.success:
  ; Success case
  mov   eax, 1              ; sys_exit = 1
  mov   ebx, 0              ; exit code 0 (success)
  int   0x80                ; make system call
```

---

## System Call Reference

### Common Linux System Calls (32-bit):

- **sys_exit (1)**: Terminate program
  - `eax = 1`, `ebx = exit_code`
- **sys_read (3)**: Read from file
  - `eax = 3`, `ebx = file_descriptor`, `ecx = buffer`, `edx = count`
- **sys_write (4)**: Write to file
  - `eax = 4`, `ebx = file_descriptor`, `ecx = buffer`, `edx = count`
- **sys_open (5)**: Open file
  - `eax = 5`, `ebx = filename`, `ecx = flags`, `edx = mode`
- **sys_close (6)**: Close file
  - `eax = 6`, `ebx = file_descriptor`
- **sys_creat (8)**: Create file
  - `eax = 8`, `ebx = filename`, `ecx = mode`

### Notes:

- System calls are invoked using the `int 0x80` instruction
- System call numbers are placed in the `eax` register
- Arguments are passed in `ebx`, `ecx`, `edx`, `esi`, `edi` (in that order)
- Return values are typically in `eax`
- File descriptors: 0 = stdin, 1 = stdout, 2 = stderr
