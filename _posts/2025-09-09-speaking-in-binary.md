---
layout: post
title: "Speaking in Binary"
date: 2025-09-09
---

*An invisible contract*

Do you know how one piece of code talks to another? 

When you write a program in a compiled language, sometimes you need to run code that someone else compiled. Sometimes. Probably it doesn't happen very often... 

For example, if you call `printf()` in C, you're invoking code that was compiled years ago, mabye by a different compiler than yours, on a different machine, by people you've never met. Every time you link against a library or make a system call, two independently compiled pieces of binary code are having a conversation.

Obviously, stuff like this happens all the time. Every program you run is a patchwork of code from different sources: your application code, the standard library, system libraries, third-party dependencies. They all speak to each other fluently, despite being strangers.

How does this work? When we learn programming top-down, we think in comfortable abstractions: classes, functions, parameters, return values. But at the CPU level, there are no functions. There are only instructions and memory addresses. There's no calling or returning in the way we normally think of it. Only jumping to different locations in memory. You don't "pass arguments" to functions like they are people. To answer this question we'll have to move down the tech stack to a level at which the abstraction of functions doesn't even exist and learn about the application binary interface (ABI).

## What does a function look like in assembly?

Let's start with the simplest possible function. One that adds two numbers:

```c
int add(int a, int b) {
    return a + b;
}
```

Here's what this actually looks like in x86-64 assembly:

```asm
add:
    movl %edi, %eax    # Move first argument to return register
    addl %esi, %eax    # Add second argument to it
    ret                # Jump back to caller
```

If you haven't seen assembly before this won't immediately make sense, but here's what you need to know: Assembly programs contain instructions (like `movl` and `addl`) that tell the CPU what to do, operands (the things being operated on), and labels (like `add:`) that mark locations in the code. You might also see directives that start with a period (like `.global` or `.section`) which are instructions for the assembler itself, not the CPU. The `movl` instruction copies data from one place to another. The things that start with `%` like `%edi` and `%eax` are registers. These are physical locations inside the CPU in which variables can be stored and accessed with extremely low latency. There are only about 16 of them in the x86 architecture, and they each have conventional uses.

So that's it. There are three instructions. No "function" construct, no "parameters" or "return statement" in the way we usually think about them. Let's break down what's really happening:

1. **Arguments arrive in registers**: The caller put the first argument in `%edi` and the second in `%esi`. At this level, passing variables is a physical process and it consists mainly of just agreeing where to leave values. But who decided on this convention? On x86-64 Linux, the System V ABI dictates that the first integer argument goes in `%rdi` (or `%edi` for 32-bit values), the second in `%rsi`/`%esi`, the third in `%rdx`/`%edx`, and so on. Different operating systems use different conventions. If the caller and callee don't agree on this convention, your program crashes.

2. **Computation happens**: We move the first value to `%eax` (which will be our return register) and add the second value to it.

3. **"Returning" is just jumping**: The `ret` instruction pops an address off the stack and jumps there. That address was pushed by whoever called us.

When someone calls this function:

```asm
movl $5, %edi      # Put 5 in first argument register
movl $3, %esi      # Put 3 in second argument register  
call add           # Push return address and jump to 'add'
# Return here with result in %eax (will be 8)
```

The `call` instruction does two things: pushes the address of the next instruction onto the stack (so we know where to return), then jumps to the `add` label. The `ret` instruction does the opposite: pops that address and jumps back.

## The Prologue and Epilogue

Our simple `add` function was too basic to need local variables or to call other functions. But what happens when a function needs its own workspace?

Let's look at a function that actually needs some breathing room:

```c
int calculate(int x, int y) {
    int temp1 = x * 2;
    int temp2 = y * 3;
    return temp1 + temp2;
}
```

In assembly, this becomes:

```asm
calculate:
    # PROLOGUE - Set up our workspace
    pushq %rbp          # Save the caller's base pointer
    movq %rsp, %rbp     # Establish our own base pointer
    subq $16, %rsp      # Allocate 16 bytes for local variables
    
    # Function body
    movl %edi, -4(%rbp)  # Store x in local variable
    movl %esi, -8(%rbp)  # Store y in local variable
    
    movl -4(%rbp), %eax  # Load x
    addl %eax, %eax      # Multiply by 2 (x + x)
    movl %eax, -12(%rbp) # Store temp1
    
    movl -8(%rbp), %eax  # Load y  
    leal (%eax,%eax,2), %eax  # Multiply by 3
    movl %eax, -16(%rbp) # Store temp2
    
    movl -12(%rbp), %eax # Load temp1
    addl -16(%rbp), %eax # Add temp2
    
    # EPILOGUE - Clean up and leave
    movq %rbp, %rsp     # Restore stack pointer
    popq %rbp           # Restore caller's base pointer
    ret
```

Every non-trivial function performs this same prologue and epilogue. Here's why:

**The Prologue** creates a consistent workspace:
1. `pushq %rbp` - Save where the caller was working (like a bookmark)
2. `movq %rsp, %rbp` - Mark where OUR workspace begins
3. `subq $16, %rsp` - Reserve space for our local variables

Now we have a "stack frame"—our own private workspace. The base pointer (`%rbp`) marks the top of our frame, and we access our locals using negative offsets from it: `-4(%rbp)`, `-8(%rbp)`, etc.

**The Epilogue** undoes everything in reverse:
1. `movq %rbp, %rsp` - Throw away our workspace
2. `popq %rbp` - Restore the caller's bookmark
3. `ret` - Jump back to caller

This creates a beautiful property: each `%rbp` points to the previous `%rbp`, forming a chain. If you're deep in nested function calls, you can walk this chain all the way back to main(). This is exactly how debuggers show you stack traces:

```
#0  calculate() at example.c:5
#1  process() at example.c:12  
#2  main() at example.c:20
```

The debugger is literally following the breadcrumbs of saved `%rbp` values.

Without this standard pattern, functions would corrupt each other's data. With it, every function gets isolated workspace that automatically cleans up when the function returns. It's the assembly equivalent of "leave the campsite as you found it."

## Calling Conventions

We've seen that functions need to agree on where to put arguments and return values. But that's just the beginning. The full "calling convention" is like a detailed contract that specifies every aspect of how functions interact.

### System V AMD64 ABI (Linux/Unix/macOS)

This is what most of the Unix world uses:

**Argument Passing:**
- First 6 integer arguments: `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9`
- Additional arguments: Pushed onto the stack
- Return value: `%rax`

**Register Preservation:**
- **Caller-saved** (temporary, can be destroyed): `%rax`, `%rdi`, `%rsi`, and others
- **Callee-saved** (must be preserved): `%rbx`, `%rbp`, `%r12`-`%r15`, and others

If you're writing a function and you want to use `%rbx`, you'd better save it first and restore it before returning. But `%rax`? Go wild—the caller knows it might get destroyed.

### Why These Rules Matter

Imagine you're calling a function from a library. You put arguments in the "wrong" registers according to that library's expectations:

```asm
# Using wrong register for argument
movq $42, %rcx       # Oops, should be %rdi on Linux
call printf          # printf expects first arg in %rdi
# Crash or garbage output
```

The function is looking in `%rdi` for its first argument, but you put it in `%rcx`. At best, it prints garbage. At worst, it interprets whatever random value was in `%rdi` as a pointer and crashes.

The ABI also requires 16-byte stack alignment before function calls for performance reasons. These conventions are why you need different compilers for different platforms, and why inline assembly is so dangerous—you're taking responsibility for following all these rules manually.

## The Invisible Infrastructure

Application Binary Interfaces are critical to how any modern computer operates. Yet, they are now so deeply buried by higher level abstractions that most programmers will never come into contact with them. It's a fundamental convention that's been so successful that most people are not even aware of its existence. The thing I find fascinating about them is that it's a contract that's so low level that it becomes physical. A carefully choreographed dance of filing and arrangement that allows compiled code to operate together.

Right now, as you read this, your computer is executing millions of function calls per second, each one following these conventions perfectly. Your browser calling the operating system, which calls device drivers, which call firmware. Code compiled decades ago successfully calling code compiled yesterday. The ABI makes it all possible, a universal language spoken in binary, following rules so consistent that we can link code together without ever seeing its source.

And that's how code talks to code.

*This article was written with a mostly written directly with Claude Code and some quite opinionated prompting. Hopefully the article is as educational for you as it is for me!*