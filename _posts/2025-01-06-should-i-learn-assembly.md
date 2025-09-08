---
layout: post
title: "Should I learn assembly? (Part 1)"
date: 2025-09-06
---

*A Python programmer's descent into the machine*

Well, I already made the decision. I am learning assembly! Is this a good use of my time? As an ML engineer working mostly in python, I'll explain some reasons why this is a bad idea. But first, some background.

## What is assembly?

Assembly is a family of very low level programming languages that map closely to machine instructions that a CPU will receive and execute. The most widely known variants of assembly are x86 (used by Intel and AMD), and ARM assembly.

These variants are known as instruction set architectures, and they are specific to the hardware on which they run. This is the level at which programming languages interface with the silicon inside the CPU on whatever device you're reading this. Many of the instructions which make up these languages represent logical operations that move data between registers and memory locations.

Although learning assembly will not tell you much about how a CPU works internally, it's getting fairly close to that boundary.

## You should not learn assembly.

### It's abstracted away

Today the majority of developers will barely ever see, let alone write, a single line of assembly. Instead we build applications at a higher level, most likely using an interpreted language, such as JavaScript, Java or Python, that doesn't even map directly to assembly instructions. Whether we are working on a user interface, a backend service, or a database, the idea of moving data physically in and out of registers in a running CPU is mostly not relevant. Instead we may think more in terms of the document object model (DOM) of a webpage, a database schema or an API contract.

The ability to express and operate with these concepts in programming languages is a marvel. I think most people are pretty happy working at that high level where they can turn their ideas more easily into useful applications.

### The world is moving the other way

Assembly code is made of simple instructions, but getting anything interesting done with it is complex. Given the effort it would take to learn, wouldn't there be more value in learning something at a higher level? This has been the trend at least since the invention of the first compiler.

Today the trend continues and is clearly moving towards programming with the assistance of AI. More lines of code are being generated; fewer lines of code are being read. Great things can be achieved very quickly but naturally there are drawbacks in this approach.

AI assistants are great for greenfield projects, but they struggle with large entangled codebases, and even the most powerful agents still routinely write quite bad code. Basically, they don't know how to carry out complex projects and they need guidance and guardrails. So instead we move towards developing those guardrails: spec-driven development, better planning, automated code review, etc. As of today this trend is likely to continue.

### Assembly code might be boring

As stated above, it's very hard to do anything interesting with it. And what's interesting? Visual feedback is interesting. Making an impact on people's lives is interesting.

The problem is that almost nobody writes applications in assembly anymore. There are some fun ideas of course. You can learn assembly by writing a sorting algorithm, or maybe something more complicated like a pong game in the terminal, but are you really going to do anything useful with it? Most likely not, and if the only things you'll be able to write in any reasonable timeframe are simple educational projects or recreations of things that a compiler can do better anyway, then what's the point?

## What next?

All told, we now have a pretty compelling case to not learn assembly! Why bother?

If I want to build an application I don't need it. I don't need to know what an electron is either. Somebody should know, but real progress is mostly made by standing on the shoulders of giants. You should do (at least) one thing well. And _writing_ assembly is probably not going to be it.

But in part 2, we'll see what counter-arguments we can bring. 

*This article was mostly not written with AI, other than for proof-reading and some rubber-ducking.*
