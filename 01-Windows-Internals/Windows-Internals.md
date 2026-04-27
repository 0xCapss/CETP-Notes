---
tags:
  - cetp
  - windows-internals
  - PE
  - user-mode
  - kernel-mode
phase: cetp
date:
status: todo
lo: LO-01, LO-02, LO-03
---
# Module I — Windows Internals

## Learning Objectives

- [ ] 
- [ ] 
- [ ] 
## Key Concepts

### Process
#### User-mode Presentation - Process Properties
##### General information
- A **process** is a container that separate application from each other.
- It can manages **Threads**, **Handles**, **Token** and **Memory**.
	- A **Thread** is a smallest sequence of programmed instruction that can be managed by a scheduler. It exists within a process and shares its memory space with other threads of the same process. Threads are scheduled for execution by the operating system and managed in the context of a process. 
	- A **Handle** is an abstract reference to a kernel object (file, process, thread, registry key, ...) that allows user-mode code to interact with kernel resources without direct memory access. 
	- A **Token** is a kernel object that describes the security context of a process  or thread (user identity, group memberships, privileges).
- However, a **process** doesn't run any code, only the **Thread** can run the code.
- We can identify a process by its Process ID (**PID**), not with it's executable file.
- If a process is destroyed, another process can reuse its **PID**
- Every process has an **integrity** level which defines what the process is able to do. 
##### Integrity level
- **Low integrity:**
	- The least trust level. Can **only** interact with **low-integrity** process.
	- It can only write to specific low-integrity location such as: ***%USERPROFILE%\AppData\LocalLow*** folder 
### PE Format

### Windows Architecture

### Execution Flow


## Lab

### Goal

### Steps

### Result

## Code / Tools

```c

```

## Offensive Relevance

> Why does this matter for a Red Teamer?

## Detection & Mitigations

> How would a Blue Team detect this?
