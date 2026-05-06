---
tags:
  - cetp
  - windows-internals
  - PE
  - user-mode
  - kernel-mode
status: todo
lo: LO-01, LO-02, LO-03
---
# Module I. Windows Internals. PE Format

## Learning Objectives

- [ ] Identify the main components of a PE file and their purpose.
- [ ] Explain the role of the DOS Header, NT Header, Section Headers, and Sections.
- [ ] Understand how RVAs work and how they relate to memory mapping.
- [ ] Describe the Import and Export directories and their offensive relevance.
- [ ] Explain why section characteristics matter for shellcode execution.

---
## Key Concepts

### PE Format

A PE (Portable Executable) is the file format used for executables on Windows. Common extensions include `.exe`, `.dll`, `.sys`, and `.scr`. Every PE file follows the same structure, composed of a series of headers followed by sections containing the actual code and data.

![PE diagram|96](../assets/1-PE-Diagram.png)

Each header is a data structure that holds metadata about the file. The headers describe how the operating system should load and execute the file in memory.

---
### DOS Header

The DOS Header occupies the first 64 bytes of the file. Its first two bytes, called `e_magic`, always hold the value `4D 5A`, which represents the ASCII string `MZ`. This signature is used to verify that the file is a valid PE. The last four bytes of the DOS Header, called `e_lfanew`, indicate the offset of the NT Header and are always located at offset `0x3C`.

---
### DOS Stub

The DOS Stub is a small piece of code placed just after the DOS Header. It is not a header but is worth knowing. When the file is executed in a DOS environment, it prints the message "This program cannot be run in DOS mode." It serves no purpose on modern Windows systems and is often replaced or removed in custom PE files.

---
### NT Header

The NT Header starts with the four-byte signature `50 45 00 00`, which represents `PE\0\0` in ASCII.

It contains two sub-structures. The **File Header** holds properties such as the target architecture and the number of sections. The **Optional Header**, despite its name, is mandatory for executable images and contains the following key fields.

The `Magic` field indicates whether the image is 32-bit (`0x10B`) or 64-bit (`0x20B`). The `AddressOfEntryPoint` field holds the Relative Virtual Address (RVA) of the entry point, which is where execution begins. The `SizeOfImage` field indicates the total size of the PE file when loaded into memory.

The last member of the Optional Header is the **Data Directory**, an array of `IMAGE_DATA_DIRECTORY` structures. Each entry points to a specific table inside the PE. The two most relevant entries are the Export Directory at index 0, which lists functions and variables exported by the executable, and the Import Directory at index 1, which lists functions imported from other executables.

> **Offensive relevance.** The Import Directory is the primary target of PEB walking and IAT hooking. Shellcodes parse the Export Directory of `kernel32.dll` to resolve WinAPI addresses dynamically without relying on static imports, which would be visible to static analysis tools.

---
### Section Headers

Section headers form an array of `IMAGE_SECTION_HEADER` structures. Each entry describes one section of the executable and contains the following fields. `SizeOfRawData` indicates the size of the section on disk. `VirtualSize` indicates the size of the section when loaded in memory. `VirtualAddress` holds the RVA of the section in memory. `Characteristics` defines the memory access rights for that section, such as Read, Read-Write, or Read-Execute.

---
### Sections

Sections contain the actual code and data of the executable. Each section has a standardized name and a specific purpose.

`.text` is the first section and contains the executable code. It is marked as Read-Execute (RX).

`.rdata` contains read-only initialized data such as string literals and constants. It is marked as Read (R).

`.data` contains initialized global and static variables that the program may modify during execution. It is marked as Read-Write (RW).

`.reloc` contains the relocation table, which is used when the image cannot be loaded at its preferred base address. It is marked as Read (R).

`.rsrc` is the resource section and contains embedded resources such as icons and images.

> **Offensive relevance.** Section characteristics directly determine what memory protection is applied when the PE is loaded. A section marked as RWX is immediately suspicious to EDRs. Shellcode loaders that inject code into a new section typically allocate it as RW, write the payload, then switch to RX via `VirtualProtect` before execution to reduce the detection surface.

---
## Offensive Relevance Summary

|Concept|Technique|Goal|
|---|---|---|
|Import Directory|IAT hooking|Intercept or redirect API calls|
|Export Directory|PEB walking|Resolve WinAPI addresses without static imports|
|Section characteristics|RW then RX allocation|Execute shellcode with lower EDR visibility|
|`AddressOfEntryPoint`|Entry point patching|Redirect execution flow at load time|
|`SizeOfImage`|PE injection|Allocate correct memory size in target process|

---
## Detection and Mitigations

|Technique|Detection|Tool or Event|
|---|---|---|
|IAT hooking|Unusual function pointers in the IAT|EDR memory scanning, PE-bear|
|PEB walking|Shellcode iterating the module list|ETW, EDR behavioral analysis|
|RWX section|Section with RWX characteristics|Static analysis, EDR memory scanning|
|Entry point patching|Modified `AddressOfEntryPoint` at runtime|EDR integrity checks|

---
## References

- [Microsoft, PE Format](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
- [ired.team, Import Address Table](https://www.ired.team/offensive-security/code-injection-process-injection/import-adress-table-iat-hooking)
- [ired.team, PEB walking](https://www.ired.team/offensive-security/defense-evasion/finding-kernel32-base-and-function-addresses-in-shellcode)
- [MITRE T1055, Process Injection](https://attack.mitre.org/techniques/T1055/)
- [MITRE T1574, Hijack Execution Flow](https://attack.mitre.org/techniques/T1574/)