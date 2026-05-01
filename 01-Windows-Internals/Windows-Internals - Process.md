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

### User-mode Presentation - Process Properties
#### General information
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
#### Integrity level
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
#### Process protection levels
- These levels are defined by **2 components:**
	- **Signer**: who signed the process (Windows, Antimalware).
	- **Protection type**: Strength of the protection (Protected or Protected-Light).

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
#### Mitigation policies
- **Protection mitigation policies** are security mechanisms implemented in Windows **to reduce the attack surface** and **protect against exploitation techniques**. We will see some examples below.
- **Data Execution Prevention (DEP)**: Prevents code execution from non-executable memory region (stack, heap). Block buffer overflow exploit.
- **Address Space Layout Randomisation (ASLR)**: Randomizes memory address of executables, DLLs, to mitigate Return-Oriented Programming (ROP) and code reuse attacks.
- **Control Flow Guard (CFG):** Prevent memory corruption exploits.
- **Signature restricted**: Allow only Microsoft-signed code to execute.
- **Image Restricted**: Blocks loading executables/DLLs from network share (SMB, WebDav).

### User-mode Presentation - Process Components
#### Threads
- Executes codes allowing concurrent execution of tasks.
- It exists **3 different types** :
	- **Main Thread**: Execute the main function.
	- **Threads create by code** (ex: *CreateThread*): Executes specific functions or tasks defined by the programmer, running concurrently with the main thread and other threads.
	- **Worker Thread**: perform cleanups, resource management, ...
- **3 States**:
	- **Waiting**: The thread is paused, waiting an event or condition before it continue execution.
	- **Ready**: The thread is prepared to execute but it's waiting for processor availability.
	- **Running:** The thread is running code on a processor.
- **Access mode**:
	- **User mode:** The thread operates with **limited privileges**, interacting with user-space memory and executing user code. It uses the **user mode stack.**
	- **Kernel mode**: The thread operate with **elevated privileges**, allowing direct interaction with hardware and system resources. When a thread switch between this 2 modes (during a syscall for example), it begins to use the **kernel mode stack.**
- **Thread stacks**:
	- **User mode stack**: 
		- Resides in the process's user space. 
		- Used for local variable, function parameters, and return address during execution.
		- Grows dynamically as needed.
	- **Kernel mode stack**:
		- Resides in kernel stack.
		- Smaller and fixed-size (12kB on 32-bit system, 24kB on 64-bit system)
- **Thread call stacks:** list of function calls that show the path the program took to reach the current point of execution.
#### Table of Handles
- Kernel can expose different types of objects (directory, files, ...) for use by user mode process accessed through Handles.
- Every process has a private handle table to kernel objects.
- Each handle can be define with a **unique value, a type, a name and the access mask.**
![[1-Windows-Internals-Tables of handles.png]]
#### Token
- The access token stores **the default security context** of process:
	- **User account and groups** the process belongs to. It defines what resources it can access based on its memberships and restrictions.
	- **Privileges** granted to the process, indicating what action the process can perform.
![[2-Windows Internals-Token.png]]
#### Virtual memory
- Virtual memory address in x64 user is 128TB relative to each process starting from **0x00000000 00000000 to 0x0007FFF FFFFFFFF.**
- Virtual memory pages by order:
	- USER_SHARED_DATA
	- PEB(Process Environment Block)
	- Stacks
	- Heap
	- NLS & MUI Files
	- Allocated Pages
	- Executable File image
	- Modules
![[3-WIndows Internals-Virtual memory.png]]
- Virtual memory pages can have **3 states:**
	- **Free**: Pages are not allocated.
	- **Commited**: All allocated pages.
	- **Reserved**: Pages with reserved address spaces for future use, not allocated yet.
- Virtual memory pages can have **3 types:**
	- **Mapped**: Pages associated with files on disk.
	- **Private**: Pages exclusive to a process.
	- **# Module I - Windows Internals - Process

## Key Concepts

### User-mode Presentation - Process Properties
#### General information
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
#### Integrity level
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
#### Process protection levels
- These levels are defined by **2 components:**
	- **Signer**: who signed the process (Windows, Antimalware).
	- **Protection type**: Strength of the protection (Protected or Protected-Light).

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
#### Mitigation policies
- **Protection mitigation policies** are security mechanisms implemented in Windows **to reduce the attack surface** and **protect against exploitation techniques**. We will see some examples below.
- **Data Execution Prevention (DEP)**: Prevents code execution from non-executable memory region (stack, heap). Block buffer overflow exploit.
- **Address Space Layout Randomisation (ASLR)**: Randomizes memory address of executables, DLLs, to mitigate Return-Oriented Programming (ROP) and code reuse attacks.
- **Control Flow Guard (CFG):** Prevent memory corruption exploits.
- **Signature restricted**: Allow only Microsoft-signed code to execute.
- **Image Restricted**: Blocks loading executables/DLLs from network share (SMB, WebDav).

### User-mode Presentation - Process Components
#### Threads
- Executes codes allowing concurrent execution of tasks.
- It exists **3 different types** :
	- **Main Thread**: Execute the main function.
	- **Threads create by code** (ex: *CreateThread*): Executes specific functions or tasks defined by the programmer, running concurrently with the main thread and other threads.
	- **Worker Thread**: perform cleanups, resource management, ...
- **3 States**:
	- **Waiting**: The thread is paused, waiting an event or condition before it continue execution.
	- **Ready**: The thread is prepared to execute but it's waiting for processor availability.
	- **Running:** The thread is running code on a processor.
- **Access mode**:
	- **User mode:** The thread operates with **limited privileges**, interacting with user-space memory and executing user code. It uses the **user mode stack.**
	- **Kernel mode**: The thread operate with **elevated privileges**, allowing direct interaction with hardware and system resources. When a thread switch between this 2 modes (during a syscall for example), it begins to use the **kernel mode stack.**
- **Thread stacks**:
	- **User mode stack**: 
		- Resides in the process's user space. 
		- Used for local variable, function parameters, and return address during execution.
		- Grows dynamically as needed.
	- **Kernel mode stack**:
		- Resides in kernel stack.
		- Smaller and fixed-size (12kB on 32-bit system, 24kB on 64-bit system)
- **Thread call stacks:** list of function calls that show the path the program took to reach the current point of execution.
#### Table of Handles
- Kernel can expose different types of objects (directory, files, ...) for use by user mode process accessed through Handles.
- Every process has a private handle table to kernel objects.
- Each handle can be define with a **unique value, a type, a name and the access mask.**
![[1-Windows-Internals-Tables of handles.png]]
#### Token
- The access token stores **the default security context** of process:
	- **User account and groups** the process belongs to. It defines what resources it can access based on its memberships and restrictions.
	- **Privileges** granted to the process, indicating what action the process can perform.
![[2-Windows Internals-Token.png]]
#### Virtual memory
- Virtual memory address in x64 user is 128TB relative to each process starting from **0x00000000 00000000 to 0x0007FFF FFFFFFFF.**
- Virtual memory pages by order:
	- USER_SHARED_DATA
	- PEB(Process Environment Block)
	- Stacks
	- Heap
	- NLS & MUI Files
	- Allocated Pages
	- Executable File image
	- Modules
![[3-WIndows Internals-Virtual memory.png]]
- Virtual memory pages can have **3 states:**
	- **Free**: Pages are not allocated.
	- **Commited**: All allocated pages.
	- **Reserved**: Pages with reserved address spaces for future use, not allocated yet.
- Virtual memory pages can have **3 types:**
	- **Mapped**: Pages associated with files on disk.
	- **Private**: Pages exclusive to a process.
	- **Image**: Pages from executable files (EXE, dll, ...)
- Virtual Memory Pages by **protection**:
	- **R** --> Read
	- **RW** --> Read and Write
	- **X** --> Execute
	- **RX** --> Read and Execute
	- **RWX** --> Read, Write, and Execute
	- **No Access** --> **Any access attempt** (read, write, execute) triggers an `STATUS_ACCESS_VIOLATION`
	- **Guard** --> This is a **modifier flag** (combined with another protection, e.g. `PAGE_READWRITE | PAGE_GUARD`)
## References
- https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/overview-of-threat-mitigations-in-windows-10
- https://learn.microsoft.com/en-us/windows/win32/api/_processthreadsapi/
- https://medium.com/@derakhshanfar.hossein/all-you-need-to-know-about-virtual-memory-page-tables-page-faults-for-backend-engineers-0fdb4b6d1565
**: Pages from executable files (EXE, dll, ...)
- Virtual Memory Pages by **protection**:
	- R --> Read
	- RW --> Read and Write
	- X --> Execute
	- RX --> Read and Execute
	- RWX --> Read, Write, and Execute
	- No Access --> **Any access attempt** (read, write, execute) triggers an `STATUS_ACCESS_VIOLATION`
	- Guard --> This is a **modifier flag** (combined with another protection, e.g. `PAGE_READWRITE | PAGE_GUARD`)
## References
- https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/overview-of-threat-mitigations-in-windows-10
- https://learn.microsoft.com/en-us/windows/win32/api/_processthreadsapi/
- https://medium.com/@derakhshanfar.hossein/all-you-need-to-know-about-virtual-memory-page-tables-page-faults-for-backend-engineers-0fdb4b6d1565
