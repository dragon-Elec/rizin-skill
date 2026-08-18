# Debugger

Debuggers are IO plugins. Spawn/attach/control via URIs. `rizin -L` — plugins with `d` in first column support debugging (e.g. `debug`, `gdb`, `winkd`, `qnx`, `ptrace`). Backends: GNU/Linux, Windows, macOS, BSDs, Solaris; also MIPS/PPC/RISC-V/ARM via gdbserver.

## Startup / attach
```
rizin -d /bin/ls        # spawn, native debugger, does NOT run yet
rizin -d PID            # attach
rizin -d gdb://host:port   # remote gdbserver (or -D gdb)
rizin -a arm -b 16 -d gdb://192.168.1.43:9090
rizin -d "windbg://..."   # Windows DbgEng backend (see windows.md)
ood                     # reopen current file in debug mode (ood args...)
```
- Rizin forks, pauses early in `ld.so` (entrypoint + libs not yet visible). Use `dcu entry0`, or set `dbg.bep=entry|main` in rizinrc.
- Attach to running pid: `rizin -d <pid>` or `rizin ptrace://pid` (io-only, no backend hook).
- Restart variants under `oo` (debug: `ood`).

## Execution control
```
dc        continue (run)
dcu addr  continue until addr (e.g. dcu main / entry0)
dcs       continue until syscall; dcs* trace all syscalls
dcr       continue until return
dcb       continue backwards (reverse debug)
dcf       debug until fork (then dp to pick process)
ds        step into; ds N  step N
dso       step over
dsu addr  step until addr; dsi step until condition
dr eip=...  set instruction pointer (skip code)
```
Function keys (visual): F2 toggle bp, F4 run-to-cursor, F7 step in, F8 step over, F9 continue.

## Breakpoints (`db`)
- `db addr|fcn` add; `db- addr` remove; `db` list. `dbl` list (alias).
- `dbc 'cmd' @ addr` — run a rizin command on breakpoint (conditional/automation). Example: `dbc 'pxw 1 @ esp-0xc' @ 0x80484d6`.
- `dbW <msg> <class|handle>` — Windows window-message breakpoints (windows.md).
- Reverse: set bp, then `dcb` returns to it (revdebug).
- Hardware regs (Intel): manipulate DR0-DR7 via `dr` for hardware breakpoints.

## Registers (`dr`)
- `dr` — all GPRs + flags; `dr rip`/`dr eax` get; `dr eax=33` / `dr rip=esp` set.
- `drr` — register references/telescoping (like peda). `dro` — old register state. `drd` — diff (old vs new). `drp file` — restore from saved.
- `dr*` — output as rizin commands (save: `dr* > regs.saved`, restore `drp`). Useful for snapshot save/restore.
- `dr eflags=pst` / `=azsti` — set flag bits. `dr=` concise view. `dr?` help.
- Register names usable in expressions/seek (`s rsp+0x40`, `px @ eip`).

## Stack
- `px 64 @ rsp` / `pxw @ esp` — view stack. `pxw rbp-rsp @ rsp` — frame state.
- `dbt` — backtrace; `dbtj` json. `dbg.btdepth`, `dbg.btalgo` tune.
- `dbt @t` / `~* k`-style — backtrace all threads (`thread apply all bt` equiv).

## Memory maps (`dm`)
- `dm` list maps; `dm=` ascii bars; `dm.` current map; `dmm` modules (libraries loaded); `dmS [lib [sect]]` sections of a lib.
- `dmi lib symbol` — find symbol in loaded lib (e.g. `dmi libc system` → address for ROP). `dmi.` closest symbol.
- `dm+ size` allocate at offset; `dm-` deallocate; `dmp perms [size]` change page perms; `dmL` allocate + huge page; `dmd[aw]` dump map regions to files; `dml file` load file into map.
- `dmh` — glibc heap map; `dmhg` heap graph; `dmhd` bins (tcache/fast/unsorted/small/large); `dmhb`, `dmhf`, `dmhi`, `dmhm`, `dmht`. Supports Glibc, Jemalloc<5, Windows heap. `dmw` Windows heap, `dmx` jemalloc.

## Processes / threads (`dp`)
- `dp` list/attach threads/processes; `dpa`/`dp=` attach; `dp-` detach.
- `dbg.follow.child` — follow fork child (default follows parent).

## Misc
- `dk <signal>` kill signal (`dk 9`). `dk[lnNo]` signals.
- `dg file` — generate core dump. `dd` — debug file descriptors (like lsof; seek/close/dup; can swap stdio for sockets). `de` ESIL watchpoints. `dx[aers]` code injection. `di` debug info. `dl` debug handler.
- `dw <pid>` block prompt until pid dies. `dW` Windows process commands.
- `R!` — pass command to backend (e.g. `R! help`, `R! ptrace`, `R! pid`, `R! mem`). `R!?` per plugin.

## Reverse debugging
- `dts+` start recording state; `dts` list sessions (diff-style, memory-space efficient); `dsb` step back; `dcb` continue back to breakpoint; `dtst file` export; `dtsf file` import. Notes per record.
- ESIL-mode reverse: `aets+` record, `aesb` step back.
- Remote reverse: `R!dsb`/`R!dcb` on a gdbserver that supports it (e.g. Mozilla rr); `R! rd` checks availability.

## Inspection after breakpoint — workflow
1. `dc` → hits bp → prompt at bp address.
2. `pd 5 @ eip` / `pdf` — current code. `dr` regs. `px @ rsp` stack.
3. `dm.` current map; `dmm` which lib; `axt`/`axf` context.
4. `dr eip=next` to skip, `dc` to continue, or `dso`/`ds` to trace.

## Gotchas
- Analysis not automatic in debug — run `aaa` (or use `-A`).
- In debug mode, writes affect memory only, not the on-disk file.
- `dcu main` may run pre-main code (constructors/TLS) — caution with malware.
- Some targets lack hardware single-step (e.g. MIPS) — rizin uses analysis+software bps; behavior may differ.
- `ood` reopens in debug after having analyzed; combine with `dor` to set env (`dor setenv=FOO=BAR`).
- Cross-debugger features (attach/detach/step tables for IDA/GDB/WinDbg) → `source/debugger/migration.md`.
- Full getting-started and memory-map walkthroughs → `source/debugger/*.md`.
