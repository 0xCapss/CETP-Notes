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
# Module I - Windows Internals - Process

## Key Concepts

### Process
#### User-mode Presentation - Process Properties
##### General information
- A **process** is a container that separate application from each other.
- It can manages **Threads**, **Handles**, **Token** and **Memory**.
	- A **Thread** is a smallest sequence of programmed instruction that can be managed by a scheduler. It exists within a process and shares its memory space with other threads of the same process. Threads are scheduled for execution by the operating system and managed in the context of a process. 
	- A **Handle** is an abstract reference to a kernel object (file, process, thread, registry key, ...) that allows user-mode code to interact with kernel resources without direct memory access. 
	- A **Token** is a kernel object that describes the security context of a process  or thread (user identity, group memberships, privileges).
- Each process has its **own address space**, its **own threads**, its **own handle table**, its **own token**, its own **PID**
- However, a **process** doesn't run any code, only the **Thread** can run the code.
- We can identify a process by its Process ID (**PID**), not with it's executable file.
- If a process is destroyed, another process can reuse its **PID**.
- Every process has an **integrity** level which defines what the process is able to do. 
##### Integrity level
- **Low integrity:**
	- The least trust level. Can **only** interact with **low-integrity** process.
	- It can only write to specific low-integrity location such as: ***%USERPROFILE%\AppData\LocalLow*** folder and limited access to **HKEY_CURRENT_USER**. 
- **Medium integrity:**
	- Default level for the most process (Microsoft Word, ...).  It can interact with **Medium** or **Low** integrity process. 
	- Can write only on **user-owned** folders like Documents, Downloads, Desktop or AppData/Roaming and access only to **HKEY_CURRENT_USER** registry.
- **High Integrity:**
	- This level describes for applications running with **Administrative privileges**. Can interact with **High, Medium and Low** integrity process.
	- Can write to System directories (C:\Windows or Program Files) and full access to registry keys.
- **System Integrity:**
	- The **highest level** integrity. It can interact with all integrity level.
	- It has **full access** to the file system and the registry key.
##### Process protection levels
- These levels are defined by **2 components:**
	- **Signer**: who signed the process (Windows, Antimalware).
	- Protection type: Strength of the protection (Protected or Protected-Light).

- **PS_PROTECTED_SYSTEM - 0x72**
	- Signer: WinSystem(7)
	- Protection: Protected(2)
	- Purpose: Higher protection for hypervisor-related process (ex: to secure the Kernel).
- **PS_PROTECTED_LSA_LIGHT - 0x41**
	- Signer: LSA (4)
	- Protection: Protected-Light (1)
	- Purpose: Shield the Local Security Authority (LSASS) & Prevent Credentials Attacks (e.g Mimikatz).
- **PS_PROTECTED_ANTIMALWARE_LIGHT - 0x31**
	- Signer: Antimalware (3)
	- Protection: Protected-Light (1)
	- Purpose: Secure antivirus/antimalware software from tampering.
##### Mitigation policies
- **Protection mitigation policies** are security mechanisms implemented in Windows **to reduce the attack surface** and **protect against exploitation techniques**. We will see some examples below.
- **Data Execution Prevention (DEP)**: Prevents code execution from non-executable memory region (stack, heap). Block buffer overflow exploit.
- Adress Space Layout Randomisation (ASLR): 
## References
