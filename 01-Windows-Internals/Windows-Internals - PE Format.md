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
# Module I - Windows Internals - PE Format

## Key Concepts

### PE Format
A PE (Portable executable) is a file format for executables for Windows. A few example of PE file extensions are `.exe`, `.dll`, `.sys` and `.src`. 

The diagram shows a simplified structure of a PE file. Every header is defined as a data structure that hold information about the PE file. 

![PE diagram](../assets/1-PE-Diagram.png)
### Dos header
- It contains the 64 bytes of the file.
- First 2 bytes called **e_magic** it's value are always **4D 5A** which represent "**MZ**" the **DOS Header signature.** 


## References
