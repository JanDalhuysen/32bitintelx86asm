# 32-bit Intel x86 Assembly Examples

This document contains various examples of 32-bit Intel x86 assembly programs using NASM syntax. These examples are designed for open-book tests and practical exercises.

## Table of Contents

1. [Basic Arithmetic Functions](#basic-arithmetic-functions)
2. [Loop-Based Functions](#loop-based-functions)
3. [Recursive Functions](#recursive-functions)
4. [Pointer and Array Functions](#pointer-and-array-functions)
5. [String Functions](#string-functions)
6. [Mathematical Functions](#mathematical-functions)

---

## Basic Arithmetic Functions

### Maximum of Two Numbers (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int max(int x, int y); - cdecl calling convention
;*****************************************************************************
global max
extern printf

%define y [ebp+12]    ; Second parameter (y)
%define x [ebp+8]     ; First parameter (x)

max:
  push  ebp         ; Save old base pointer
  mov   ebp, esp    ; Set up new base pointer

  mov   eax, x      ; Load x into eax
  cmp   eax, y      ; Compare x and y
  jg    .return_x   ; Jump if x > y
  mov   eax, y      ; If y >= x, return y
  jmp   .end
.return_x:
  mov   eax, x      ; If x > y, return x
.end:
  mov   esp, ebp    ; Restore stack pointer
  pop   ebp         ; Restore base pointer
  ret               ; Return (eax contains result)
```

### Minimum of Two Numbers

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int min(int x, int y); - No specific calling convention
;*****************************************************************************
global min

%define y [ebp+12]
%define x [ebp+8]

min:
  push  ebp
  mov   ebp, esp

  mov   eax, x      ; Load x into eax
  cmp   eax, y      ; Compare x and y
  jl    .return_x   ; Jump if x < y
  mov   eax, y      ; If y <= x, return y
  jmp   .end
.return_x:
  mov   eax, x      ; If x < y, return x
.end:
  mov   esp, ebp
  pop   ebp
  ret
```

### Absolute Value

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int abs(int x); - cdecl calling convention
;*****************************************************************************
global abs

%define x [ebp+8]

abs:
  push  ebp
  mov   ebp, esp

  mov   eax, x      ; Load x into eax
  cmp   eax, 0      ; Compare x with 0
  jge   .positive   ; If x >= 0, it's already positive
  neg   eax         ; If x < 0, negate it (make positive)
  jmp   .end
.positive:
  ; eax already contains the positive value
.end:
  mov   esp, ebp
  pop   ebp
  ret
```

---

## Loop-Based Functions

### Power Function (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int power(int x, int n); - cdecl calling convention
;*****************************************************************************
global power

%define n [ebp+12]    ; Exponent
%define x [ebp+8]     ; Base

power:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save ebx (callee-saved register)
  push  ecx         ; Save ecx (we'll use it as counter)

  mov   eax, 1      ; Initialize result to 1
  mov   ebx, x      ; Load base into ebx
  mov   ecx, n      ; Load exponent into ecx

  ; Handle negative exponents (return 0 for integer division)
  cmp   ecx, 0
  jl    .negative_exp

.loop:
  cmp   ecx, 0      ; Check if exponent is 0
  je    .done       ; If yes, we're done
  imul  eax, ebx    ; Multiply result by base
  dec   ecx         ; Decrement exponent
  jmp   .loop       ; Continue loop

.negative_exp:
  mov   eax, 0      ; For negative exponents, return 0 in integer context
  jmp   .done

.done:
  pop   ecx         ; Restore ecx
  pop   ebx         ; Restore ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### Fast Power Function (Binary Exponentiation)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int fast_power(int x, int n); - Binary exponentiation
;*****************************************************************************
global fast_power

%define n [ebp+12]
%define x [ebp+8]

fast_power:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save registers
  push  ecx

  mov   eax, 1      ; Initialize result to 1
  mov   ebx, x      ; Load base
  mov   ecx, n      ; Load exponent

.loop:
  test  ecx, ecx    ; Check if exponent is 0
  jz    .done       ; If yes, we're done

  test  ecx, 1      ; Check if exponent is odd
  jz    .even       ; If even, skip multiplication

  imul  eax, ebx    ; Multiply result by current base

.even:
  imul  ebx, ebx    ; Square the base
  shr   ecx, 1      ; Divide exponent by 2
  jmp   .loop

.done:
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### Factorial Function

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int factorial(int n); - cdecl calling convention
;*****************************************************************************
global factorial

%define n [ebp+8]

factorial:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save ebx for counter

  mov   eax, 1      ; Initialize result to 1
  mov   ebx, n      ; Load n into ebx (our counter)

  ; Handle base cases: factorial(0) = 1, factorial(1) = 1
  cmp   ebx, 1
  jle   .done

.loop:
  cmp   ebx, 1      ; Check if counter <= 1
  jle   .done       ; If yes, we're done
  imul  eax, ebx    ; Multiply result by current counter
  dec   ebx         ; Decrement counter
  jmp   .loop       ; Continue loop

.done:
  pop   ebx         ; Restore ebx
  mov   esp, ebp
  pop   ebp
  ret
```

---

## Recursive Functions

### Greatest Common Divisor (GCD) - Iterative Version

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int gcd(int a, int b); - Euclidean algorithm (iterative)
;*****************************************************************************
global gcd

%define b [ebp+12]
%define a [ebp+8]

gcd:
  push  ebp
  mov   ebp, esp

  mov   eax, a      ; Load a into eax
  mov   ecx, b      ; Load b into ecx

.loop:
  test  ecx, ecx    ; Check if b == 0
  jz    .end        ; If yes, return a

  xor   edx, edx    ; Clear edx for division
  div   ecx         ; eax = eax / ecx, remainder in edx
  mov   eax, ecx    ; a = b
  mov   ecx, edx    ; b = remainder
  jmp   .loop

.end:
  mov   esp, ebp
  pop   ebp
  ret
```

### Ackermann Function (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int ackermann(int m, int n); - cdecl calling convention
;*****************************************************************************
global ackermann

%define n [ebp+12]
%define m [ebp+8]

ackermann:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save ebx for temporary calculations

  mov   eax, m      ; Load m into eax
  cmp   eax, 0      ; Check if m == 0
  jne   .m_not_zero ; If not, go to m_not_zero
  ; If m == 0: return n + 1
  mov   eax, n      ; Load n
  inc   eax         ; Increment by 1
  jmp   .done

.m_not_zero:
  mov   eax, n      ; Load n
  cmp   eax, 0      ; Check if n == 0
  jne   .n_not_zero ; If not, go to n_not_zero
  ; If n == 0: return ackermann(m-1, 1)
  push  dword [ebp+12]  ; Push original n (will be ignored, but for clarity)
  push  dword [ebp+8]   ; Push original m
  dec   dword [esp]     ; Decrement m (m-1)
  push  1               ; Push 1
  call  ackermann       ; Call ackermann(m-1, 1)
  add   esp, 8          ; Clean up stack (2 parameters)
  jmp   .done

.n_not_zero:
  ; If n != 0: return ackermann(m-1, ackermann(m, n-1))
  push  dword [ebp+12]  ; Push original n
  push  dword [ebp+8]   ; Push original m
  dec   dword [esp]     ; Decrement m (m-1)
  push  dword [ebp+12]  ; Push original n
  dec   dword [esp]     ; Decrement n (n-1)
  push  dword [ebp+8]   ; Push original m again
  call  ackermann       ; Call ackermann(m, n-1)
  add   esp, 8          ; Clean up stack for inner call
  push  eax             ; Push result of inner call
  call  ackermann       ; Call ackermann(m-1, result)
  add   esp, 8          ; Clean up stack for outer call

.done:
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

---

## Pointer and Array Functions

### Swap Two Integers (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* void swap(int *x, int *y); - cdecl calling convention
;*****************************************************************************
global swap

%define x_ptr [ebp+8]   ; Pointer to first integer
%define y_ptr [ebp+12]  ; Pointer to second integer

swap:
  push  ebp
  mov   ebp, esp
  push  edi           ; Save registers
  push  esi

  mov   edi, x_ptr    ; Load address of x
  mov   esi, y_ptr    ; Load address of y

  mov   eax, [edi]    ; Load value at x into eax
  xchg  eax, [esi]    ; Exchange eax with value at y
  xchg  eax, [edi]    ; Exchange eax with value at x (complete swap)

  pop   esi           ; Restore registers
  pop   edi
  mov   esp, ebp
  pop   ebp
  ret
```

### Array Sum (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int array_sum(int arr[], int size); - cdecl calling convention
;*****************************************************************************
global array_sum

%define arr_ptr [ebp+8]   ; Pointer to array
%define size    [ebp+12]  ; Size of array

array_sum:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  ecx
  push  esi

  mov   eax, 0        ; Initialize sum to 0
  mov   ebx, arr_ptr  ; Load array pointer
  mov   ecx, size     ; Load array size
  mov   esi, 0        ; Initialize index to 0

.loop:
  cmp   esi, ecx      ; Compare index with size
  jge   .done         ; If index >= size, exit loop

  mov   edx, [ebx + esi*4]  ; Load arr[index] (4 bytes per int)
  add   eax, edx            ; Add to sum
  inc   esi                 ; Increment index
  jmp   .loop

.done:
  pop   esi           ; Restore registers
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### Linear Search

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int linear_search(int arr[], int size, int target); - cdecl calling convention
;*****************************************************************************
global linear_search

%define arr_ptr  [ebp+8]   ; Pointer to array
%define size     [ebp+12]  ; Size of array
%define target   [ebp+16]  ; Target value to find

linear_search:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  ecx
  push  edx
  push  esi

  mov   ebx, arr_ptr  ; Load array pointer
  mov   ecx, size     ; Load array size
  mov   edx, target   ; Load target value
  mov   esi, 0        ; Initialize index to 0

.loop:
  cmp   esi, ecx      ; Compare index with size
  jge   .not_found    ; If index >= size, element not found

  mov   eax, [ebx + esi*4]  ; Load arr[index]
  cmp   eax, edx            ; Compare with target
  je    .found              ; If equal, we found it

  inc   esi                 ; Increment index
  jmp   .loop

.found:
  mov   eax, esi            ; Return index where found
  jmp   .done

.not_found:
  mov   eax, -1             ; Return -1 if not found

.done:
  pop   esi                 ; Restore registers
  pop   edx
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

---

## String Functions

### String Length (cdecl)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int strlen(const char *str); - cdecl calling convention
;*****************************************************************************
global strlen

%define str_ptr [ebp+8]   ; Pointer to string

strlen:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  esi

  mov   ebx, str_ptr  ; Load string pointer
  mov   esi, 0        ; Initialize counter to 0

.loop:
  mov   al, [ebx + esi]   ; Load character at str[counter]
  cmp   al, 0             ; Check if it's null terminator
  je    .done             ; If yes, we're done
  inc   esi               ; Increment counter
  jmp   .loop

.done:
  mov   eax, esi          ; Return length
  pop   esi               ; Restore registers
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### String Copy

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* void strcpy(char *dest, const char *src); - cdecl calling convention
;*****************************************************************************
global strcpy

%define dest_ptr [ebp+8]  ; Destination string pointer
%define src_ptr  [ebp+12] ; Source string pointer

strcpy:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  ecx
  push  esi
  push  edi

  mov   edi, dest_ptr ; Load destination pointer
  mov   esi, src_ptr  ; Load source pointer
  mov   ebx, 0        ; Initialize index to 0

.loop:
  mov   al, [esi + ebx]   ; Load character from source
  mov   [edi + ebx], al   ; Store character to destination
  cmp   al, 0             ; Check if it's null terminator
  je    .done             ; If yes, we're done copying
  inc   ebx               ; Increment index
  jmp   .loop

.done:
  pop   edi               ; Restore registers
  pop   esi
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### String Compare

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int strcmp(const char *str1, const char *str2); - cdecl calling convention
;*****************************************************************************
global strcmp

%define str1_ptr [ebp+8]  ; First string pointer
%define str2_ptr [ebp+12] ; Second string pointer

strcmp:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  ecx
  push  edx
  push  esi

  mov   ebx, str1_ptr ; Load first string pointer
  mov   ecx, str2_ptr ; Load second string pointer
  mov   esi, 0        ; Initialize index to 0

.loop:
  mov   al, [ebx + esi]   ; Load character from str1
  mov   dl, [ecx + esi]   ; Load character from str2
  cmp   al, dl            ; Compare characters
  jne   .different        ; If different, we can return
  cmp   al, 0             ; Check if we reached end of string
  je    .equal            ; If yes, strings are equal
  inc   esi               ; Increment index
  jmp   .loop

.different:
  cmp   al, dl            ; Compare the differing characters
  jl    .str1_smaller     ; If str1[char] < str2[char], str1 is smaller
  mov   eax, 1            ; If str1[char] > str2[char], return 1
  jmp   .done
.str1_smaller:
  mov   eax, -1           ; If str1[char] < str2[char], return -1
  jmp   .done

.equal:
  mov   eax, 0            ; Strings are equal

.done:
  pop   esi               ; Restore registers
  pop   edx
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

---

## Mathematical Functions

### Fibonacci Sequence (Iterative)

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int fibonacci(int n); - Iterative fibonacci calculation
;*****************************************************************************
global fibonacci

%define n [ebp+8]

fibonacci:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save registers
  push  ecx

  mov   eax, n      ; Load n
  cmp   eax, 1      ; Check if n <= 1
  jg    .calculate  ; If n > 1, calculate
  ; If n <= 1, return n (fibonacci(0) = 0, fibonacci(1) = 1)
  jmp   .done

.calculate:
  mov   ebx, 0      ; Initialize fib(n-2) = 0
  mov   ecx, 1      ; Initialize fib(n-1) = 1
  mov   edx, 2      ; Initialize current index = 2

.loop:
  cmp   edx, n      ; Compare current index with n
  jg    .done       ; If index > n, we're done

  mov   eax, ebx    ; Load fib(n-2)
  add   eax, ecx    ; Add fib(n-1) to get fib(n)
  mov   ebx, ecx    ; Move fib(n-1) to fib(n-2)
  mov   ecx, eax    ; Move fib(n) to fib(n-1)
  inc   edx         ; Increment index
  jmp   .loop

.done:
  ; Result is in eax
  pop   ecx         ; Restore registers
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### Prime Number Check

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int is_prime(int n); - Check if a number is prime
;*****************************************************************************
global is_prime

%define n [ebp+8]

is_prime:
  push  ebp
  mov   ebp, esp
  push  ebx         ; Save registers
  push  ecx

  mov   eax, n      ; Load n
  cmp   eax, 2      ; Check if n < 2
  jl    .not_prime  ; If n < 2, not prime
  cmp   eax, 2      ; Check if n == 2
  je    .prime      ; If n == 2, it's prime
  test  eax, 1      ; Check if n is even
  jz    .not_prime  ; If even and > 2, not prime

  ; Check odd divisors from 3 to sqrt(n)
  mov   ebx, 3      ; Start with divisor 3
  mov   ecx, eax    ; Copy n to ecx
  shr   ecx, 1      ; Upper bound = n/2 (simplified)

.loop:
  cmp   ebx, ecx    ; Compare divisor with upper bound
  jg    .prime      ; If divisor > upper bound, it's prime

  mov   edx, 0      ; Clear edx for division
  mov   eax, n      ; Load n
  div   ebx         ; Divide n by current divisor
  cmp   edx, 0      ; Check if remainder is 0
  je    .not_prime  ; If remainder is 0, not prime

  add   ebx, 2      ; Increment divisor by 2 (check only odd numbers)
  jmp   .loop

.prime:
  mov   eax, 1      ; Return 1 (true)
  jmp   .done
.not_prime:
  mov   eax, 0      ; Return 0 (false)

.done:
  pop   ecx         ; Restore registers
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

### Binary Search in Sorted Array

```nasm
; asmsyntax=nasm
;*****************************************************************************
;* int binary_search(int arr[], int size, int target); - cdecl calling convention
;*****************************************************************************
global binary_search

%define arr_ptr [ebp+8]   ; Pointer to array
%define size    [ebp+12]  ; Size of array
%define target  [ebp+16]  ; Target value to find

binary_search:
  push  ebp
  mov   ebp, esp
  push  ebx           ; Save registers
  push  ecx           ; low
  push  edx           ; high
  push  esi           ; mid

  mov   ebx, arr_ptr  ; Load array pointer
  mov   ecx, 0        ; Initialize low = 0
  mov   edx, [ebp+12] ; Load size
  dec   edx           ; high = size - 1

.search_loop:
  cmp   ecx, edx      ; Compare low with high
  jg    .not_found    ; If low > high, element not found

  mov   eax, ecx      ; Load low
  add   eax, edx      ; Add high
  shr   eax, 1        ; Divide by 2 to get mid
  mov   esi, eax      ; Store mid in esi

  ; Calculate address of arr[mid]
  mov   eax, [ebx + esi*4]  ; Load arr[mid]
  cmp   eax, [ebp+16]       ; Compare with target
  je    .found              ; If equal, we found it
  jl    .search_right       ; If arr[mid] < target, search right half
  ; Else search left half
  mov   edx, esi      ; high = mid - 1
  dec   edx
  jmp   .search_loop

.search_right:
  mov   ecx, esi      ; low = mid + 1
  inc   ecx
  jmp   .search_loop

.found:
  mov   eax, esi      ; Return index where found
  jmp   .done

.not_found:
  mov   eax, -1       ; Return -1 if not found

.done:
  pop   esi           ; Restore registers
  pop   edx
  pop   ecx
  pop   ebx
  mov   esp, ebp
  pop   ebp
  ret
```

---

## Notes

### Key Points about 32-bit x86 Assembly:

1. **Calling Conventions**:
   - `cdecl` is the standard calling convention where the caller cleans up the stack
   - Parameters are pushed right-to-left onto the stack
   - Return values are placed in the `eax` register

2. **Stack Frame Setup**:

   ```nasm
   push  ebp         ; Save old base pointer
   mov   ebp, esp    ; Set up new base pointer
   ```

   - `ebp+8` is the first parameter
   - `ebp+12` is the second parameter
   - And so on...

3. **Register Usage**:
   - `eax`: Return value register
   - `ebx`, `esi`, `edi`: Callee-saved registers (must be preserved)
   - `ecx`, `edx`: Caller-saved registers (can be modified)

4. **Memory Addressing**:
   - `[ebp+8]`: Access parameter at offset 8 from base pointer
   - `[ebx + esi*4]`: Access array element (4 bytes per integer)

5. **Common Instructions**:
   - `mov`: Move data
   - `cmp`: Compare values
   - `je`, `jl`, `jg`, `jle`, `jge`: Conditional jumps
   - `imul`: Signed integer multiplication
   - `div`: Unsigned integer division
   - `push`/`pop`: Stack operations
