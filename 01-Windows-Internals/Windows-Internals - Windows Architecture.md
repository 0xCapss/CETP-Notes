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
# Module I - Windows-Internals - Windows Architecture

## Key Concepts
### Windows Architecture
A processor inside a machine running Windows can operate in 2 modes: **User mode** and **kernel mode**. Applications run in user mode and OS in kernel mode. When an application wants to accomplish a task, like create a file, it cannot do it on its own. The only entity that can complete the task is the kernel, so instead applications must follow a specific function call flow. The diagram below shows a high level of this flow.

![](1-Windows%20Architecture.png)

1. **User process**: A program/Application is executed by the user (Notepad, Word, ...)
2. **Subsytem DLLs**: DLLs contain API function that are called by user process. An example can be `kernel32.dll`, exporting for example the *CreateFile* function from Windows API. Other common subsystem DLLs are `ntdll.dll`, `advapi32.dll`, and `user32.dll`.
3. **Ntdll.dll**: A system-wide DLL which is the lowest layer available in user mode. This is a special DLL that creates the transition from user mode to kernel mode.
4. **Executive kernel**: This is known as the **Windows Kernel.** It calls other drivers and module available with kernel mode to complete tasks.  The Windows kernel is partially stored in a file called `ntoskrnl.exe` under "C:\Windows\System32".
## References
