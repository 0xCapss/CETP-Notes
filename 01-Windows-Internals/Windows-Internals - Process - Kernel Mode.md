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
- It contains 128 TB for System 