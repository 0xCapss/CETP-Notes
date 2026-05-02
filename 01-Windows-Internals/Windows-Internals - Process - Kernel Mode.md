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
# Module I - Windows Internals - Process - Kernel Mode

## Key Concepts
### Kernel-mode presentation
#### General
- **Kernel**: Piece of code that's running into memory. It has many goals:
	- **Manage** the actual processes that are running on the system.
	- Make sure they **have separated memory space.**
	- Manage the privilege between those processes.
	- It enforce security barriers between processes.
- It contains 128 TB for System Address Space, **0xFFFF8000 00000000 to 0xFFFFFFFF FFFFFFFF**.
- Kernel tracks the state of the process during execution using *_EPROCESS* structure.
- *_EPROCESS* is a kernel memory structure that describe a process.
	- Image name, token, protection level, memory, ...
- *_EPROCESS* structure change across Windows versions.
#### EPROCESS Overview
- To get process’s EPROCESS address:
``