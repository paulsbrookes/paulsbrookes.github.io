---
layout: post
title: "Contrast Learning: x86 vs ARM"
date: 2025-09-08
---

*Understanding computer architecture through differences*

When learning a complex field, the hardest part isn't just finding the right information; it's knowing what deserves attention. This is especially true for topics like assembly language and CPU architecture which are far removed from the layer of the tech stack in which I normally work. As someone entering this world without intending to become a hardware engineer, it would be good to have a learning strategy that would allow me to focus my effort.

## The Learning Goal

My goal is broad: I want to understand how computers work at a very low level. Not to design chips, but to build a mental model that helps me reason about what I observe in higher-level programming. When I see performance characteristics in compiled languages, when I see a segfault (¯\_(ツ)_/¯), when I want to interface with the operating system, I want intuition about what's happening beneath the abstractions.

## The Learning Approach: Understanding Through Contrast

Rather than trying to exhaustively learn one architecture, I'm comparing two different approaches at a higher level: x86 and ARM. This comparison serves as a useful learning tool for several reasons:

**Differences reveal design decisions.** Where x86 and ARM diverge, we see that their designers made explicit choices. These aren't arbitrary; they represent different trade-offs and priorities. Understanding why people disagree may tell you more than memorizing what everyone accepts.

**Contrasts highlight boundaries.** When you see two systems handle the same problem differently, you start to see the joints of the problem space. The interfaces between components become clearer. These won't be the only way of breaking down the problem, but with some decades of expertise and work going into them, they are worth examining.

**Disagreement is information-dense.** The places where architectures differ are where the interesting engineering lives. Where they agree might be necessary fundamentals, but where they disagree reveals the genuine trade-offs in computer design.

So let's examine three key architectural differences between x86 and ARM, not to say which is better, but to understand what these differences teach us about how computers really work.

## Key Difference #1: How the ALU Accesses Memory

This might be the most fundamental split between x86 and ARM architectures.

**x86: Direct Memory Operations**

In x86, the arithmetic logic unit (ALU) can work directly with memory. Want to add 5 to a value stored in RAM? One instruction:

```asm
ADD [memory_address], 5
```

The processor fetches from memory, performs the addition, and writes back, all in a single instruction. You can even do complex operations like:

```asm
XOR EAX, [EBX + ESI*4 + 0x1000]
```

This XORs a register with a value from memory, using a complex addressing calculation to find the memory location.

**ARM: Load-Store Architecture**

ARM takes a different approach. The ALU can only operate on values already in registers. To perform that same "add 5 to memory" operation requires three instructions:

```asm
LDR R1, [memory_address]  ; Load from memory to register
ADD R1, R1, #5            ; Add 5 to the register
STR R1, [memory_address]  ; Store back to memory
```

**What This Teaches Us**

This difference cascades through the entire architecture. x86's approach means fewer instructions but more complex hardware; the processor needs circuits that can handle memory access within arithmetic operations. ARM's approach means simpler hardware but more instructions.

The trade-off here is between instruction count and predictability. x86 saves instructions but creates variable timing (memory access is slow and unpredictable). ARM uses more instructions but each has predictable timing, making it easier to optimize and pipeline.

## Key Difference #2: Instruction Encoding

The two architectures differ significantly in how they encode instructions.

**x86: Variable Length (1-15 bytes)**

x86 instructions vary in length based on what they need to encode. A simple `NOP` (no operation) is just one byte. But a complex instruction like:

```asm
MOV [EBX + ESI*4 + 0x12345678], 0x87654321
```

That's 11+ bytes; it needs room for the opcode, the addressing mode, the 32-bit displacement, and the 32-bit immediate value.

**ARM: Fixed Length**

ARM instructions have fixed sizes:
- Classic ARM mode: every instruction is exactly 32 bits (4 bytes)
- Thumb mode: every instruction is exactly 16 bits (2 bytes)

No exceptions. A simple `NOP` takes the same space as the most complex operation the architecture allows.

**What This Teaches Us**

This trade-off has significant implications. Variable-length encoding saves space for simple operations but adds complexity: the processor doesn't know where the next instruction starts until it decodes the current one. You have to parse as you go.

Fixed-length encoding uses more space (32 bits even for simple operations) but gains simplicity. The processor always knows instruction boundaries. It can fetch and decode multiple instructions in parallel without waiting. This choice affects everything from power consumption to pipeline design.

Modern x86 processors translate their variable-length instructions into fixed-length micro-operations internally, showing that the fixed-length approach has advantages.

## Key Difference #3: Conditional Execution

x86 and ARM handle conditional logic differently at the assembly level.

**x86: Branch Around Unwanted Code**

x86 handles conditionals primarily through branching. Check a condition, jump if needed:

```asm
CMP EAX, 0        ; Compare EAX with 0
JNE skip          ; Jump if not equal
MOV EBX, 5        ; This only executes if EAX == 0
skip:
ADD ECX, 1        ; Execution continues here
```

Every conditional requires predicting whether the branch will be taken. Wrong predictions flush the pipeline, which is expensive on modern processors.

**ARM: Predicated Instructions**

ARM (in 32-bit mode) allows most instructions to be conditional. Instead of jumping around code, you mark instructions to execute only under certain conditions:

```asm
CMP R0, #0        ; Compare R0 with 0
MOVEQ R1, #5      ; Move ONLY if equal (EQ)
ADDNE R2, R2, #1  ; Add ONLY if not equal (NE)
SUBGT R3, R3, #2  ; Subtract ONLY if greater than (GT)
```

No branches. No pipeline flushes. The instructions flow through normally; they just don't commit their results if their condition isn't met.

**What This Teaches Us**

This reveals different philosophies about handling uncertainty. x86 relies on branch prediction; modern x86 processors have sophisticated branch predictors, maintaining history tables and pattern recognition. It's a complex solution to a problem created by the branch-heavy approach.

ARM's conditional execution eliminates the problem for simple cases. But it has costs: every instruction needs bits to encode the condition, and complex out-of-order processors struggle with conditional instructions. 64-bit ARM (AArch64) removed most conditional execution, suggesting that for high-performance out-of-order execution, branches with good prediction might be better.

## Overall

These three differences (memory access, instruction encoding, and conditional execution) aren't just technical details. They represent fundamental questions in computer architecture:

1. **Simplicity vs. Capability**: Should instructions be simple (easier hardware) or powerful (fewer instructions)?

2. **Predictability vs. Flexibility**: Is it better to have consistent, predictable behavior or flexible, adaptive behavior?

3. **Hardware vs. Software Complexity**: Should complexity live in the hardware (x86's approach) or in the software/compiler (ARM's approach)?

But, after highlighting all these differences are both architectures are converging? Modern ARM chips add complexity for performance. Modern x86 chips translate to RISC-like micro-ops internally. The market may be pushing both architectures to a new middle ground.

*This article was written with a pretty heavy mix of AI generation and human editing. After a deep-dive learning conversation with Claude, the resulting key points were translated into this article before being heavily edited into a form I like.*