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
```cmd
!process 0 0 explorer.exe
```
- To get process’s EPROCESS fields:
```cmd
dt nt!_eprocess <EPROCESS_addr>
```
+ Interesting fields:
	- UniqueProcessId(PID)
	- ImageFileName => Process Comparaison
	- Token => Privilege Escalation
	* Protection => Bypass PPL
	- ActiveProcessLinks => Enumerate processes
	- InheritedFromUniqueProcessId (PPID) => Identify the parent
	- Peb
	- ObjectTable => Stores handles to kernel objects
	- SectionBaseAddress
	- ThreadListHead => Iterate the Threads.
#### ActiveProcessLinks & Process Hiding
- Double-linked list that list all the process in the OS together.
![List of process](4-Windows-Internal-List-process.png)
- Each process has its own **ActiveProcessLinks** that contains **Flink** and **Blink**.
- Flink is pointing on the **ActiveProcessLinks** of the **next process**.
- Blink is pointing on the **ActiveProcessLinks** of the **previous process**.
**![ActiveProcessLinks](5-Windows-internals-ActiveProcessLinks.png)
- To hide cmd process, we have to unlink its **ActiveProcessLinks** from that chain by making its previous process **ActiveProcessLinks**’s Flink pointing to the **cmd next process ActiveProcessLinks.**
- the cmd next process ActiveProcessLinks’s Blink pointing to the cmd previous process’s ActiveProcessLinks.
![Hide cmd](6-Windows-Internals-Hide-cmd.png)
#### Token
