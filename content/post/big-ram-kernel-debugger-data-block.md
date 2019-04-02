+++
title = "Big RAM and the kernel debugger data block"
description = "Running into problems with large RAM dumps not being parsed by Volatility or Rekall? It happened to me, too"
tags = [
    "memory",
    "volatility",
    "rekall",
    "windows",
]
date = "2017-04-28"
categories = [
    "DFIR",
]
author = "Dan"
summary = true 
+++

I recently was provided with a 64GB raw memory image created by the free DumpIt tool from 
a Windows 2012 R2 x64 server. To my dismay, neither Volatility nor Rekall could make heads 
or tails of it. After debugging a little further, I discovered that neither tool was able 
to find the kernel debugger data block (`_KDDEBUGGER_DATA64`) structure.

After a lot of Googling and consultation of the handy bible on this subject matter, 
[*The Art of Memory Forensics*](http://a.co/azFkE04), I think I finally understand what 
went wrong (but I’m eagerly seeking clarification/correction from the DFIR/infosec 
community if I’m mistaken!).

When Volatility attempts to make sense of a Windows 64-bit RAM dump, they use the global 
variable `PsActiveProcessHead`, which is a pointer to the kernel’s list of `_EPROCESS` 
structures (and other global variables such as `PsLoadedModuleList`, which is a pointer 
to the kernel’s list of drivers). Since these variables (symbols) are not exported by the 
kernel, they can’t simply be looked up in `ntoskrnl`. “Profiles” in tools such as 
Volatility contain hard-coded addresses where it can expect to find such variables in 
specific versions of Windows with specific patches/hotfixes applied (and so it knows which 
symbols, data structures, and algorithms to use when parsing).

Within the Kernel Processor Control Region (KPCR) data structure there is a field that 
contains a pointer to a `_DBGKD_GET_VERSION64` structure which contains a linked list of 
`_KDDEBUGGER_DATA64` structures (kernel debugger data blocks). According to [*The Art of Memory Forensics*](http://a.co/azFkE04),

>"The debugger data structure is typically located inside the NT kernel module 
(`nt!KdDebuggerDataBlock`). It contains a build string such as 
`3790.srv03_sp2_rtm.070216-1710`, numerical values that indicate the major and minor build 
numbers, and the service pack level for the target operating system.... [T]he debugger 
data block also contains pointers to the start of the active process and loaded module 
lists.... Volatility scans for the `_KDDEBUGGER_DATA64` by looking for constant values 
embedded in the structure, including the hard-coded 4-byte signature of KDBG.... 
Additionally, in some cases there may be more than one `_KDDEBUGGER_DATA64` structure 
resident in physical memory. This can happen if the target system hasn’t rebooted since 
applying a hot patch that updated some kernel files, or if the machine rebooted so quickly 
that the entire contents of RAM was not flushed.... [M]any Volatility plugins rely on 
finding the debugger data block and then walking the active process and loaded module 
lists" (pp. 62–65).

The kernel debugger data block is used by the kernel debugger so that it can find active 
processes, handles to objects, loaded modules, etc. when figuring out the cause of a 
crash. The data in this structure is relatively stable so that Windows (and tools such as 
WinDbg) can find them, so the memory addresses of many kernel variables can be readily 
identified using this structure. This makes it very handy for memory forensics.

However, the kernel debugger data block is encoded on some 64-bit versions of Windows 
Vista and later with large amounts of RAM and some tools do not properly decode it when 
creating a raw memory image (such as DumpIt). DumpIt only properly decodes this structure 
when a crashdump is taken (rather than a raw image). Takahiro Haruyama explains this in 
detail [in a blog post he wrote in 2014 entitled "64bit Big Sized RAM Image Acquisition 
Problem"](https://takahiroharuyama.github.io/blog/2014/01/07/64bit-big-size-ram-acquisition-problem/) 
(he used the term "encrypted" rather than "encoded", but the latter term is [used by 
Volatility in a blog post referencing Haruyama’s post](https://volatility-labs.blogspot.com/2014/01/the-secret-to-64-bit-windows-8-and-2012.html) 
so I’ve elected to echo that language).

As a result, if using DumpIt, you must take a crashdump rather than a raw image. As of the 
time that Haruyama wrote his post in early 2014, this was the only supported tool that he 
tested. **However, I was able to successfully acquire and analyze the RAM in AFF4 format 
using winpmem 2.1.post4 and Rekall 1.6.0 (Gotthard), respectively** (your mileage may 
vary). This is because *Rekall no longer relies on the kernel debugger data block to identify the necessary global variables/symbols*. According to [the Rekall documentation](http://www.rekall-forensic.com/docs/Manual/Plugins/Windows/#kdbgscan),

>"Rekall no longer uses the Kernel Debugger Block for analysis — instead accurate global 
symbol information are fetched from Microsoft PDB files containing debugging symbols."

It was a good learning experience and now I know to be wary of certain collection tools 
when collecting large amounts of RAM from Windows 64-bit Vista+ systems! I figured I’d 
share the lesson. I must also give 
[credit to Brian Moran as his blog post on this issue](http://www.brimorlabsblog.com/2014/01/all-memory-dumping-tools-are-not-same.html) 
was my first stop when Googling this issue and was very helpful as I researched this (he 
also ran into this issue in 2014).

**UPDATE:** UPDATE: After posting this on Twitter, [Brian shared his experience](https://twitter.com/brianjmoran/status/858030064788803584) 
with another tool that he has had success with: [Belkasoft RAM Capturer.](https://belkasoft.com/ram-capturer)

