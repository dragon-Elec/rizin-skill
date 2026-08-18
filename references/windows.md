# Windows Debugging

## Window-message breakpoints
Useful for GUI apps (set breakpoint when a window receives a message).
- `dW` — list current process windows (Handle, PID, TID, Class Name).
- `dbW <MSG> <ClassName>` or `dbW <MSG> <handle>` — breakpoint on message handler. Example: `dbW WM_KEYDOWN Edit`, `dbW WM_KEYDOWN 0x002c048a`.
- `dWi` — identify a window with the mouse (move cursor, answer prompts, returns handle/class).

## WinDbg backend for Windows (DbgEng.dll)
Rizin can use `DbgEng.dll` as debugging backend → WinDbg capabilities: dump files, local/remote user & kernel debugging.
- Loads `dbgeng.dll` from `_NT_DEBUGGER_EXTENSION_PATH` env before default search path. (Store WinDbg Preview DLLs unusable directly.)
- Usage — pass WinDbg/kd-style options (quote/escape as needed):
```
rizin -d "windbg://-remote tcp:server=Server,port=Socket"
rizin -d "windbg://MyProgram.exe \"my arg\""
rizin -d "windbg://-k net:port=<n>,key=<MyKey>"
rizin -d "windbg://-z MyDumpFile.dmp"
```
- Debug normally (`d?`) or use `R!` to talk to backend shell: `dcu <addr>` → ModLoad/breakpoint output; `R!k4` → backtrace (WinDbg `k4`).

## WinDbg KD kernel-mode (winkd plugin)
Attach to a Windows VM's kernel over serial/network. Work-in-progress.
- Serial (Win 7+): `bcdedit /debug on`, `bcdedit /dbgsettings serial debugport:1 baudrate:115200`. XP: boot.ini `/debug /debugport=COM1 /baudrate=115200`.
- Network (Win 7+): `bcdedit /debug on`, `bcdedit /dbgsettings net hostip:w.x.y.z port:n`. Win8+: `bcedit /set {globalsettings} advancedoptions true` to show advanced boot options each boot.
- VM: VMWare → named-pipe serial (server→VM); VirtualBox → host pipe (`_Host_Pipe_` + Create Pipe); QEMU: `-chardev socket,id=serial0,path=/tmp/winkd.pipe,nowait,server -serial chardev:serial0`.
- Connect:
```
rizin -a x86 -b 32 -D winkd winkd:///tmp/winkd.pipe     # socket file
rizin -D winkd winkd://\\.\pipe\com_1                    # Windows named pipe
rizin -a x86 -b 32 -d winkd://<hostip>:<port>:w.x.y.z   # network
```
- Rizin sends a breakin packet; you land on an `int3` trap. Skip it: `dr eip=eip+1; dc` twice → VM interactive again.
- `dp` list processes; `dpa`/`dp=` attach (shows process base in physical layout).

## Windows heap
`dmw[jb?]` — Windows heap commands (alongside `dmh` glibc). See `dm?`.

## Windows env / PDB
- PDB symbol download: `pdb.server` (semicolon-separated URLs), `pdb.symstore` local store, `pdb.extract=1` (cab extract, needs `cabextract`+`wget/curl`), `pdb.autoload=true` (auto-load PDBs for linked DLLs when debugging `rizin -d file.exe`).
- UNC network paths usable as symbol servers on Windows. `idp`/`idpi` PDB commands; `rz-bin -P`/`-PP`.
- DbgEng DLLs must be executable by normal users (Store Preview app ones are not).

## Gotchas
- Window-message bps are Windows-native only (`dbW`/`dW`).
- KD is still initial/WIP; GDB remote is an alternative to reach Windows kernels.
- See `source/debugger/windows_messages.md` and `source/remote_access/windbg.md` for exact sessions.
