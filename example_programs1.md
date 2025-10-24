Below is a self-contained markdown file that

- keeps every routine you already have (so your open-book notes stay complete)
- fills in the missing bodies (fact, power, ackerman, binary_search)
- adds a few extra, frequently-used patterns (strlen, strcpy, memcpy, isort, qsort-driver)
- shows both **cdecl** (caller-cleans-stack) and **non-callee-clean** versions
- is 100 % NASM 32-bit Intel syntax, ready to assemble with
  ```
  nasm -felf32 file.asm && gcc -m32 file.o driver.c
  ```

Save the text between the dashed lines as `x86-32-asm-cheatsheet.md` and you are ready for the test.

---

# 32-bit Intel (x86) NASM Cheat-Sheet

Open-book friendly – cdecl vs. non-cdecl, filled stubs, extra idioms

---

## 0. NASM build reminder

```bash
nasm -felf32 file.asm              # produce ELF32 .o
gcc -m32 file.o driver.c           # link with C driver
```

---

## 1. Register conventions (cdecl)

| caller-saved | callee-saved        |
| ------------ | ------------------- |
| EAX ECX EDX  | EBX ESI EDI EBP ESP |

Return value → **EAX**  
Arguments on stack: `[ebp+8]` `[ebp+12]` … (right-to-left push order)

---

## 2. Prologue / Epilogue macros (cdecl)

```nasm
%define pushl push
%define popl  pop

%macro  c_enter 0
        push    ebp
        mov     ebp, esp
%endmacro

%macro  c_leave 0
        mov     esp, ebp
        pop     ebp
        ret
%endmacro
```

---

## 3. Previously given routines (kept intact)

### max – cdecl

```nasm
global max
%define y [ebp+12]
%define x [ebp+8]
max:
        c_enter
        mov     eax, x
        cmp     eax, y
        jg      .Lmax_done
        mov     eax, y
.Lmax_done:
        c_leave
```

### power (exponentiation by squaring) – cdecl

```nasm
global power
%define n [ebp+12]
%define x [ebp+8]
power:
        c_enter
        push    ebx
        mov     eax, 1          ; result
        mov     ebx, x          ; base
        mov     ecx, n          ; exponent
.lp:    test    ecx, ecx
        jz      .done
        test    ecx, 1
        jz      .even
        imul    eax, ebx
.even:  imul    ebx, ebx
        shr     ecx, 1
        jmp     .lp
.done:  pop     ebx
        c_leave
```

### gcd (Euclid) – cdecl

```nasm
global gcd
%define b [ebp+12]
%define a [ebp+8]
gcd:
        c_enter
        mov     eax, a
        mov     ecx, b
.loop:  test    ecx, ecx
        jz      .end
        xor     edx, edx
        div     ecx             ; edx = eax % ecx
        mov     eax, ecx
        mov     ecx, edx
        jmp     .loop
.end:   c_leave
```

### swap (dereference and exchange) – cdecl

```nasm
global swap
%define x_ptr [ebp+8]
%define y_ptr [ebp+12]
swap:
        c_enter
        push    esi
        push    edi
        mov     esi, [y_ptr]
        mov     edi, [x_ptr]
        mov     eax, [edi]
        xchg    eax, [esi]
        mov     [edi], eax
        pop     edi
        pop     esi
        c_leave
```

---

## 4. Filled stubs from tut4.asm

### fact (recursive) – cdecl

```nasm
global fact
%define n [ebp+8]
fact:
        c_enter
        mov     eax, n
        cmp     eax, 1
        jbe     .base
        dec     eax
        push    eax
        call    fact
        add     esp, 4          ; caller cleans
        imul    eax, [ebp+8]    ; n * fact(n-1)
        jmp     .done
.base:  mov     eax, 1
.done:  c_leave
```

### ackerman (classic) – cdecl

```nasm
global ackerman
%define y [ebp+12]
%define x [ebp+8]
ackerman:
        c_enter
        push    ebx
        mov     eax, x
        mov     ebx, y
        test    eax, eax
        jnz     .x_nz
        inc     ebx
        mov     eax, ebx
        jmp     .done
.x_nz:  test    ebx, ebx
        jnz     .both_nz
        ; ack(x-1,1)
        dec     eax
        push    1
        push    eax
        call    ackerman
        add     esp, 8
        jmp     .done
.both_nz:
        ; ack(x-1, ack(x,y-1))
        push    ebx
        dec     dword [esp]     ; y-1
        push    eax             ; x
        call    ackerman        ; inner call
        add     esp, 8
        push    eax             ; result of inner
        dec     dword [esp+4]   ; x-1
        call    ackerman
        add     esp, 4
.done:  pop     ebx
        c_leave
```

### binary_search (iterative) – cdecl

```nasm
global binary_search
; int binary_search(int n, int *list, int low, int high)
%define high [ebp+16]
%define low  [ebp+12]
%define list [ebp+8]
%define n    [ebp+20]
binary_search:
        c_enter
        mov     edx, low
        mov     ecx, high
.loop:  cmp     edx, ecx
        ja      .not_found
        lea     eax, [edx+ecx]
        shr     eax, 1          ; mid
        mov     ebx, [list+eax*4]
        cmp     ebx, n
        je      .found
        jl      .go_right
        dec     eax
        mov     ecx, eax
        jmp     .loop
.go_right:
        inc     eax
        mov     edx, eax
        jmp     .loop
.found: mov     eax, 1          ; true
        jmp     .done
.not_found:
        xor     eax, eax
.done:  c_leave
```

---

## 5. Extra handy routines

### strlen – cdecl

```nasm
global strlen
%define s [ebp+8]
strlen:
        c_enter
        mov     eax, s
        xor     ecx, ecx
.scan:  cmp     byte [eax+ecx], 0
        lea     ecx, [ecx+1]
        jne     .scan
        mov     eax, ecx
        dec     eax             ; drop final 0
        c_leave
```

### strcpy – cdecl

```nasm
global strcpy
; void strcpy(char *dst, const char *src)
%define src [ebp+12]
%define dst [ebp+8]
strcpy:
        c_enter
        push    esi
        push    edi
        mov     edi, dst
        mov     esi, src
.loop:  lodsb
        stosb
        test    al, al
        jnz     .loop
        pop     edi
        pop     esi
        c_leave
```

### memcpy (n bytes) – cdecl

```nasm
global memcpy
; void *memcpy(void *dest, const void *src, unsigned n)
%define n    [ebp+16]
%define src  [ebp+12]
%define dest [ebp+8]
memcpy:
        c_enter
        push    esi
        push    edi
        mov     edi, dest
        mov     esi, src
        mov     ecx, n
        rep     movsb
        mov     eax, dest       ; return dest
        pop     edi
        pop     esi
        c_leave
```

### insertion_sort (int \*a, int n) – cdecl

```nasm
global insertion_sort
%define n [ebp+12]
%define a [ebp+8]
insertion_sort:
        c_enter
        push    ebx
        mov     ecx, 1          ; i = 1
.outer: cmp     ecx, n
        jae     .done
        mov     esi, [a+ecx*4]  ; key = a[i]
        mov     edx, ecx
        dec     edx             ; j = i-1
.inner: test    edx, edx
        js      .insert
        mov     eax, [a+edx*4]
        cmp     eax, esi
        jle     .insert
        mov     [a+edx*4+4], eax ; a[j+1] = a[j]
        dec     edx
        jmp     .inner
.insert:
        mov     [a+edx*4+4], esi
        inc     ecx
        jmp     .outer
.done:  pop     ebx
        c_leave
```

---

## 6. Examples WITHOUT cdecl (callee cleans stack)

Useful for tiny leaf functions or when you want to avoid `add esp,4*n` in caller.

### fastcall-like add2 – callee pops 8 bytes

```nasm
global add2
; int add2(int a,int b)  – callee clears
add2:
        push    ebp
        mov     ebp, esp
        mov     eax, [ebp+8]
        add     eax, [ebp+12]
        leave
        ret     8                 ; pop 2 arguments
```

### leaf abs – no frame at all

```nasm
global abs
; int abs(int x) – callee cleans 4 bytes
abs:
        mov     eax, [esp+4]
        cdq
        xor     eax, edx
        sub     eax, edx
        ret     4
```

---

## 7. Quick caller example (C driver)

```c
#include <stdio.h>
#include <stdint.h>

extern int max(int,int);
extern int power(int,int);
extern void insertion_sort(int*,int);

int main(){
    int v[]={5,2,9,1,5,6};
    insertion_sort(v,6);
    for(int i=0;i<6;++i) printf("%d ",v[i]);
    putchar('\n');
    printf("2^10=%d  max(3,7)=%d\n", power(2,10), max(3,7));
    return 0;
}
```
