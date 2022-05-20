---
title: "Memory in Assembly - How different areas of memory are treated at the metal"
date: 2022-05-20T17:57:44
draft: false
featuredImage: /static/postimages/memory-in-assembly/ghidra.png
author: Luke Briggs
rssFullText: true
categories: [Computing, Learning]
description: A little dive into the world of stacks, heaps, and data
---

I thought it might make for an interesting learning exercise to see how memory in different areas is treated at the assembly level.
The program itself is simple, but I hope it will provide some insight.

## C Program
```c
const int constant_int = 0xc07;
int main() {
    static int static_int = 0x57a71c;
    int stack_int = 0x57ac3;
    int* malloc_int = malloc(1);
    malloc_int[0] = 0xa1;
    func_int(stack_int);
    func_int(*malloc_int);
    func_int(static_int);
    func_const(constant_int);
}
```


## ARMv8 Assembly
```objdump
stp         x29,x30,[sp, #local_20]!                
mov         x29,sp                                  
mov         w0,#0x7ac3                              
movk        w0,#0x5, LSL #16                        
str         w0,[sp, #stack_int+0x20]                
mov         x0,#0x1                                 
bl          __stubs:__stubs::_malloc                ;void * _malloc(size_t param_1)
str         x0,[sp, #malloc_int+0x20]               
ldr         x0,[sp, #malloc_int+0x20]               
mov         w1,#0xa1                                
str         w1,[x0]                                 
ldr         w0,[sp, #stack_int+0x20]                
bl          func_int                                ;void func_int(int x)
ldr         x0,[sp, #malloc_int+0x20]               
ldr         w0,[x0]                                 
bl          func_int                                ;void func_int(int x)
adrp        x0,0x100008000                          
add         x0,x0,#0x0                              
ldr         w0,[x0]=>__data:main::static_int        ;= 57A71Ch
bl          func_int                                ;void func_int(int x)
mov         w0,#0xc07                               
bl          func_const                              ;void func_const(int x)
mov         w0,#0x0                                 
ldp         x29=>local_20,x30,[sp], #0x20           
ret         
```


## Static Integer
You may notice that our static (0x57a71c) variable is not declared in the assembly. 
This is because everything you see above is in the TEXT segment. The part of the assembly that is read only. 
Text segment is labelled as such with an assembly directive.  
 
Static variables exist for the lifetime of the program but can change their value and so shouldn't be loaded into the text segment. 
Instead, values that can be edited but are static and initialised are loaded into the data segment.

```objdump
// __data
// __DATA
// ram:100008000-ram:100008003
//

_static_int.0
main::static int
    int     57A71Ch
//
//__DATA
//__DATA
// ram:100008004-ram:10000bfff
```

With Ghidra we can see that our file contains a directive to load our static variable into the data segment at launch. 
We also have a label to this static variable that somewhat matches our C code.


## Stack Integer
This works by moving our value into a register, then storing the contents of that register into an area of memory offset from the stack pointer. 
The stack pointer is a special register that the CPU uses to tell us where our stack is. 
Because everything we store on the stack is a known size, the compiler works out the offsets and knows where things should be stored in relation to each other.

### C
```c
int stack_int = 0x57ac3;
```

### Assembly

```objdump
mov         w0,#0x7ac3               ; move integer into register 0                             
movk        w0,#0x5, LSL #16                        
str         w0,[sp, #stack_int+0x20] ; store integer into RAM address pointed to by
                                     ; stack pointer (sp) + an offset based on number
                                     ; size. Our number is now in stack memory

```


## Heap Integer
This works by asking for a given amount of memory to be allocated, the kernel allocates the new memory in our address space then hands us back a pointer to it. 
The address we get back is stored on the stack, and we can load values into this address to store on the heap.

### C
```c
int* malloc_int = malloc(16);

malloc_int[0] = 0xa1;
```

### Assembly
```objdump
mov         x0,#0x1                     ; Set first parameter of malloc to 1 (number of 
                                        ; bytes to allocate)                        
bl          __stubs:__stubs::_malloc    ; call malloc
str         x0,[sp, #malloc_int+0x20]   ; Store return value of malloc (register 0)
                                        ; onto the stack (sp + offset)
 
 
 
 
ldr         x0,[sp, #malloc_int+0x20]   ; Load heap address (which is stored on stack)
                                        ; into register              
mov         w1,#0xa1                    ; move our integer into register 1              
str         w1,[x0]                     ; Store our integer into the the heap address 
                                        ; (integer is now on the heap)

```

> Notice how the bottom two lines are effectively the same as what we did on the stack, everything else is all the extra overhead of using the heap, this is one of the reasons why it's easier to use the stack to store local variables (another obvious one is we don't have to free things on the stack whereas in a real program we would also have to remember to free this memory)


## Calling a function with a stack variable

### C
```c
func_int(stack_int);
```

### Assembly
```objdump
ldr         w0,[sp, #stack_int+0x20] ; Set parameter 0 to value stored in the 
                                     ;  address pointed to by stack_int                
bl          func_int                 ; call function
```


## Calling a function with a heap variable

### C
```c
func_int(*malloc_int);
```

### Assembly
```objdump
ldr         x0,[sp, #malloc_int+0x20] ; load the value stored at the malloc_int variable (our
                                      ; heap pointer) into register 0
ldr         w0,[x0]                   ; load the value at our heap address (the actual number) 
                                      ; into register 0 as the parameter to our function.                         
bl          func_int                  ; call function
```


## Calling a function with a static variable

### C
```c
func_int(static_int);
```

### Assembly
```objdump
adrp        x0,0x100008000   ; Since static variables are in the data section,
                             ; we know the precise address (relative to our 
                             ; program) of them, and so we don't need to use the 
                             ; stack pointer, just the program counter which adrp 
                             ; does by default                        
add         x0,x0,#0x0                                    
ldr         w0,[x0]          ; load the value at the data address from memory 
                             ; into register 0 (our function parameter)
bl          func_int         ; call function

```


## Calling a function with a constant variable
```c
func_const(constant_int);
```

```objdump
mov         w0,#0xc07   ; A constant variable never changes so we don't need to 
                        ; use addresses, the constant is here in the text section 
                        ; with the code and is loaded by value into register 0
bl          func_const. ; call function
```


## Recap

- Using stack variables, we load everything relative to the stack pointer.
- With heap variables we first load the address we have on the stack, then load the value from this address
- With a static variable we load everything from a known location relative to the program counter (in the data segment)
- With a constant variable we can just use the values as is which is stored in the text of the assembly itself.

