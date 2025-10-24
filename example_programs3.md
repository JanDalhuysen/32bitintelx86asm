-----------------------------------------------------------------------
# 32-bit Linux x86 NASM – minimal syscall cheat-sheet  
( int 0x80 interface – no libc, no cdecl )

Assemble & link:  
```bash
nasm -felf32 syscall_demo.asm
ld -m elf_i386 -o syscall_demo syscall_demo.o
```

---

---

## 1. Register map (kernel expects)

```
eax  = syscall number
ebx  = arg1
ecx  = arg2
edx  = arg3
esi  = arg4
edi  = arg5
ebp  = arg6
```

Return value in **EAX** (negative ⇒ errno).

---

## 2. Common numbers (Linux 32-bit)

```nasm
%define SYS_write  4
%define SYS_read   3
%define SYS_open   5
%define SYS_close  6
%define SYS_exit   1
%define SYS_brk    45
```

---

## 3. Tiny examples

### write(1, msg, len) – “hello\n”

```nasm
global _start
section .data
msg db "hello syscall",10
len equ $-msg

section .text
_start:
    mov eax, SYS_write
    mov ebx, 1          ; stdout
    mov ecx, msg
    mov edx, len
    int 0x80            ; write

    mov eax, SYS_exit
    xor ebx, ebx        ; status 0
    int 0x80
```

### read → echo loop (max 128 B)

```nasm
section .bss
buf resb 128

section .text
global _start
_start:
    ; fd = 0 (stdin)
    mov eax, SYS_read
    xor ebx, ebx
    mov ecx, buf
    mov edx, 128
    int 0x80            ; eax = bytes read

    test eax, eax
    jbe .done           ; 0 or error

    ; echo back
    mov edx, eax        ; byte count
    mov eax, SYS_write
    mov ebx, 1
    mov ecx, buf
    int 0x80
.done:
    mov eax, SYS_exit
    xor ebx, ebx
    int 0x80
```

### open / close file

```nasm
section .data
fname db "test.txt",0
flags equ 0x42        ; O_RDWR | O_CREAT
mode  equ 0644o       ; -rw-r--r--

section .text
global _start
_start:
    ; fd = open(fname, flags, mode)
    mov eax, SYS_open
    mov ebx, fname
    mov ecx, flags
    mov edx, mode
    int 0x80
    mov esi, eax        ; save fd

    ; write(fd, "ok\n", 3)
    mov eax, SYS_write
    mov ebx, esi
    mov ecx, ok
    mov edx, 3
    int 0x80

    ; close(fd)
    mov eax, SYS_close
    mov ebx, esi
    int 0x80

    mov eax, SYS_exit
    xor ebx, ebx
    int 0x80

section .data
ok db "ok",10
```

### dynamic memory (brk)

```nasm
global _start
_start:
    ; get current break
    mov eax, SYS_brk
    xor ebx, ebx
    int 0x80        ; eax = current end

    add eax, 4096   ; ask for 1 page more
    mov ebx, eax
    mov eax, SYS_brk
    int 0x80        ; new break

    ; use memory …

    mov eax, SYS_exit
    xor ebx, ebx
    int 0x80
```

---

## 4. One-liner macros for reuse

```nasm
%macro  SYSCALL0 1
        mov eax, %1
        int 0x80
%endmacro

%macro  SYSCALL1 2
        mov eax, %1
        mov ebx, %2
        int 0x80
%endmacro

%macro  SYSCALL2 3
        mov eax, %1
        mov ebx, %2
        mov ecx, %3
        int 0x80
%endmacro
```

Example usage:

```nasm
SYSCALL2 SYS_write, 1, msg
```

---
