- Use Windbg Remote X-Mktg-X Kernel Debugging on StudentVM to change a cmd’s token to SYSTEM token.
-  First of all, we need to find the offset:
```bash
0: kd> dt !_eprocess token
nt!_EPROCESS
   +0x4b8 Token : _EX_FAST_REF
   
```
- Then, we need to find the `_EPROCESS` of a `SYSTEM` and for `cmd.exe`process via `!process 0 0`
```bash
1: kd> !process 0 0 system
PROCESS ffff968ebc89c080
    SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 006d5000  ObjectTable: ffffab034c43ff00  HandleCount: 1775.
    Image: System
```


- Use Windbg Remote X-Mktg-X Kernel Debugging on StudentVM to Hide the cmd process.
- Use Windbg Remote X-Mktg-X Kernel Debugging on StudentVM to change the notepad process protection level to PsProtectedSignerAntimalware-Light and remove the LSA protection from LSASS.
- Use Windbg Remote X-Mktg-X Kernel Debugging on StudentVM to hide a loaded driver.