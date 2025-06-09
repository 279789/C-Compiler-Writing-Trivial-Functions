<!---
{
  "id": "9264c227-8d04-4a33-bbc7-df86ba0e9a5a",
  "depends_on": ["a29aa0d7-e54c-4927-a4cc-0cd84f3b1032"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-06-05",
  "keywords": ["C Functions", "Primitive Datatypes", "Calling Conventions", "Stack Frame", "Parameter Passing"]
}
--->

# Functions with Primitive Parameters and Return Types

> In this exercise you will learn how to write and analyze C functions that use only primitive data types (such as `int`, `char`, and `float`) as parameters and return values. Furthermore we will observe how these values are passed in registers or on the stack, and how function calls are translated into machine instructions.

## Introduction

Functions are one of the most fundamental building blocks of any C program. Functions allow us to split programs into reusable, logical parts that perform specific tasks. Every function in C has a **signature**: a return type, a name, and zero or more parameters with specified types.

In this exercise, we restrict ourselves to primitive data types: `int`, `char`, and `float`. These types are directly supported by the CPU, and their function calls follow well-defined **calling conventions** that determine how arguments are passed (in registers or on the stack) and how return values are delivered.

On x86\_64 Linux, the System V ABI defines that:

* The first six integer or pointer arguments are passed in registers: `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, and `%r9`.
* Floating-point arguments are passed in `%xmm0` - `%xmm7`.
* Return values are usually delivered in `%rax` for integers, and `%xmm0` for floating-point.

> ⚠ We will avoid any structs, arrays, or complex data types here to focus purely on how primitive types are handled.

### 1.1) Further Readings and Other Sources

* [x86-64 System V ABI (PDF)](https://gitlab.com/x86-psABIs/x86-64-ABI/-/raw/master/x86-64-ABI.pdf)
* [GCC Function Calling Convention (Wiki)](https://wiki.osdev.org/System_V_ABI)
* [Beej's Guide to C Programming](https://beej.us/guide/bgc/)
* [Compiler Explorer - godbolt.org](https://godbolt.org/)

## Tasks

### Task 1: A Function with Integer Parameters and Return

Create the following file:

```c
// file: int_functions.c

#include <stdio.h>

int compute_sum(int a, int b, int c) {
    int result = (a + b) * c;
    return result;
}

int main() {
    int x = 5, y = 7, z = 3;
    int total = compute_sum(x, y, z);
    printf("Sum result: %d\n", total);
    return 0;
}
```

#### a) Compile and inspect:

* `gcc -Wall -o int_functions int_functions.c`
* `objdump -d int_functions | less`

#### b) Reflect:

* Which registers are used to pass `x`, `y`, and `z` into `compute_sum`? %edi, %esi, %edx (%edi are the lower 32 bits of %rdi)
* Where does the return value appear after `compute_sum` finishes? at %eax

---

### Task 2: A Function Using `char` Parameters

Create:

```c
// file: char_functions.c

#include <stdio.h>

char increment_chars(char a, char b) {
    char result = a + b + 1;
    return result;
}

int main() {
    char m = 'A';
    char n = 5;
    char res = increment_chars(m, n);
    printf("Char result: %d\n", res);
    return 0;
}
```

#### a) Compile:

* `gcc -Wall -o char_functions char_functions.c`

#### b) Inspect:

* Are `char` parameters still passed in full 64-bit registers?
* How is sign extension handled?
* Where is the return value stored?

---

### Task 3: A Function Using `float` Parameters

Create:

```c
// file: float_functions.c

#include <stdio.h>

float multiply_floats(float a, float b, float c) {
    return a * b * c;
}

int main() {
    float a = 1.5f, b = 2.0f, c = 3.0f;
    float result = multiply_floats(a, b, c);
    printf("Float result: %f\n", result);
    return 0;
}
```

#### a) Compile:

* `gcc -Wall -o float_functions float_functions.c`

#### b) Inspect:

* Which `%xmm` registers are used for `float` parameters?
* Where is the floating-point result returned?
* Can you observe floating-point multiplication instructions (`mulss`)?

---

### Task 4: Mixed Types

Create:

```c
// file: mixed_functions.c

#include <stdio.h>

float mixed_operation(int a, char b, float c) {
    return (a + b) * c;
}

int main() {
    int val1 = 10;
    char val2 = 3;
    float val3 = 2.5f;

    float result = mixed_operation(val1, val2, val3);
    printf("Mixed result: %f\n", result);
    return 0;
}
```

#### a) Compile:

* `gcc -Wall -o mixed_functions mixed_functions.c`

#### b) Inspect:

* How are mixed types split across general-purpose registers and floating-point registers?
* Where do you observe implicit type conversions?

---

## Questions

1. How many parameters are passed in registers on x86\_64?
2. How are `char` parameters handled inside registers?
3. Which registers are used for floating-point arguments?
4. Where do integer and floating-point return values appear after function calls?
5. Why do calling conventions exist?
6. What happens when you call functions with more than 6 parameters?
7. Why is it important for compiler, assembler, and linker to agree on calling conventions?

## Advice

Understanding how functions pass parameters and return values helps you reason about performance, debugging, and low-level programming. C allows you to write portable code while still being close to the hardware. Although you rarely need to think about calling conventions in high-level code, you absolutely must understand them when debugging assembly, reverse engineering, or writing systems code. Use `objdump` and `gdb` to verify how arguments flow into registers at runtime.
