---
name: rizin
description: Interactive binary reverse engineering with rizin — disassemble functions, find strings, inspect imports/exports, trace control flow, analyze malware, work CTF challenges, or do vulnerability research. Use when user wants to open a binary and explore it, on phrases like "open this binary", "reverse engineer", "disassemble", "analyze this binary", "debug this executable", "what does this function do", "find the flag", "find the password in this crackme", "inspect ELF/PE imports and strings", or any RE task involving an executable or library file. Covers static analysis, dynamic debugging, disassembly, patching, binary inspection of ELF/PE/Mach-O/raw/firmware, and debugging with the built-in debugger or gdb/windbg backends.
---

> **Knowledge snapshot:** Rizin v0.9.1
> **Source:** Rizin Book at `38c3bed`
>
> This skill is version-specific. Commands, flags, debugger behavior,
> and backends may change in future Rizin releases. If observed
> behavior conflicts with this skill, consult the current Rizin
> documentation/source.

# Rizin Operational Memory

Rizin = RE framework. Disassembler + hexeditor + debugger in one CLI. Treats ALL IO (files, sockets, processes, maps) as a plain file. Companion tools: `rz-bin`, `rz-asm`, `rz-ax`, `rz-hash`, `rz-diff`, `rz-find`, `rz-gg`, `rz-run`, `rz-sign`, `rz-pm`. Scriptable via `rz-pipe`.

Prompt shows current offset: `[0x...]>`. Command syntax: `[.][times][cmd][~grep][@addr][|pipe][>file] ; ...`. Append `?` to any command for help, `??` for extended. Tab-completion works.

## CLI startup
- `rizin file` — open (read-only). VA mode; starts at entry.
- `rizin -w file` — write mode (required to patch file).
- `rizin -d file` / `rizin -d PID` — debugger mode / attach.
- `rizin -A file` — run `aaa` on load (`-AA` → `aaaa`).
- `rizin -q -c 'cmd' file` — batch, quiet, run cmd then quit (`-qc`).
- `rizin =` — empty `malloc://512` buffer.
- `-a arch -b bits`, `-e k=v`, `-s addr` (seek), `-i file` (script after open), `-I file` (before open), `-n` (no bin info), `-p prj.rzdb` (project).
- `-D backend gdb://host:port` — select debug backend by URI.

## Quick cheat-sheet (strings / imports / flags)

```
iz / izz / iz~keyword      # strings in data section / all strings / search strings
px 64 @ addr / pf          # hex dump / print formatted data
ii / iE / is / il / iI     # imports / exports / symbols / linked libs / binary info
fl / f name @ addr         # list flags / set a label
CCa @ addr comment         # add comment at address
/ string / /x deadbeef     # search string in memory / hex pattern
/R pop rdi                 # search ROP gadgets matching pattern
afl~sym. / afl~main        # grep-style filter on function list
/x 55                      # 55 = push rbp (x86-64 function prologue)
```

Rizin is not a shell. Don't type `afl | grep foo` or `afl > file.txt` expecting shell
semantics. Use rizin's own operators: `~foo` grep-style filter (e.g. `afl~main`),
`~{}` JSON pretty-print, `> file` redirect (works inside rizin), `| shellcmd` pipes
to shell but quoting is fragile.

## Static-analysis workflow

Default to one-shot batch mode — faster, no race conditions, clean stdout:

```bash
rizin -A -q -N -e scr.color=0 -c 'iI; ii; izz~keyword; afl~main' /path/to/binary
```

Use `-A` (full analysis so functions/xrefs/FLIRT exist), `-q` (quit after commands,
no hang), `-c 'cmd1; cmd2; ...'` chaining with `;`, plus `-N` (ignore user config,
reproducible) and `-e scr.color=0` (no ANSI escape hell).
This is the right default for: triage/orientation (`iI`, `ii`, `iz`, `afl`), extracting
specific data to grep over, anything scriptable. Rizin can exit 0 on a bad command —
grep output for `ERROR:` and missing results.

Reserve iterative sessions (each command depends on reading the previous output —
following xrefs, renaming as you go) for genuinely interactive work.

```
iI        # binary info (arch, os, bits, canary, PIE, NX)
iS        # sections;  ii=imports  iE=exports  ie=entry  ir=relocs  iz/izz=strings
aa        # analyze all (basic). aaa deeper, aaaa experimental
afl       # list functions  (aflj json, afl~name)
pdf @ main # disassemble a function
iz | grep  # find strings, then axt to find xrefs
pd 10 @ 0x...  # disassemble N at addr
```
Search (see searching.md): `/` string, `/x hex`, `/a asm`, `/ad/ mnem`, `/w` wide, `/i` case-insens, `/m` magic, `/R` rop.

## Debugging workflow
```
rizin -d file
db @ main     # breakpoint (addr or fcn); db- remove; db list
dc            # run/continue; dcu main (until); dcs (until syscall); dcr (until return)
ds            # step into; dso step over; dsu addr step-until
dr            # registers; dr rip / dr eax=33; drr telescoped refs
px @ rsp      # stack: px 64 @ rsp / pxw @ esp
dm            # memory maps; dm= bars; dmm modules; dm. current; dmi libc system (symbol in lib)
dbt           # backtrace
dp            # threads/processes; dpa attach
dg file       # core dump
```
Break early in `ld.so` (libs not loaded) → `dcu entry0` or set `dbg.bep=entry|main`. Breakpoint commands: `dbc 'cmd' @ addr`. Conditional-ish via `dbc` + `dsi`.

## Core command families
- `s` seek (`sd`+delta, `shu/shr` undo/redo, `$$` current), `b` block size
- `f` flags / `fs` flagspaces / `fz` zones; `f- hit*` cleanup
- `p` print: `px` hex, `p8` hexpairs, `pd/pD` disasm, `pf` format, `ps` strings, `pc` C array, `pi` instr
- `w` write: `wx`, `wa` asm, `wv`, `wo` ops, `wD` de-bruijn; requires `-w`
- `c` compare (`cx`, `cc`, `cd`, `cf`)
- `a` analyze; `ax` xrefs (`axt` to, `axf` from); `t` types; `afv*` vars
- `e k=v` config (`el` list, `e k=?` values, `eco` theme)
- `% expr` math; `$` aliases; `(` macros; `@@` iterators/loops
- JSON: append `j` (`aflj`, `iIj`, `pxj`); pretty: `~{}`; column grep: `~[n]`

## Gotchas / behavior
- Analysis is NOT automatic → run `aa`/`aaa` or `-A`; `pdf` fails with "Cannot find function" if unanalyzed.
- `-w` needed to write files; in debug mode writes only affect memory (not file).
- DWARF symbols load automatically; PDB needs `pdb.autoload=true` / `pdb.server`.
- Search hits become `hit0_N` flags in `searches` space (rm: `f- hit*`). `search.in`/`analysis.in` set search/analysis ranges.
- Use `@e:k=v` for temporary config, `@ addr` for temporary seek.
- Machine-readable output (JSON) is preferable for automation; don't assume a command has it unless documented.
- For large Go/Rust binaries (20MB+), prefer `strings` + `--help` + `rz-bin` over disassembly. A 35MB Go binary has tens of thousands of `sym.func.<addr>` entries with package paths not attached to names — `afl~packagename` will match nothing even when the code is there. `strings BIN | grep -E '/api/|ENV_VAR'` and running `BIN --help` will outperform rizin for attack-surface / config questions.
- If the binary is packed/obfuscated, note it and suggest unpacking before analysis.
- Calling conventions: 32-bit x86 args on the stack; x86-64 uses RDI/RSI/RDX/RCX/R8/R9; ARM uses R0-R3; AArch64 uses X0-X7.

## Escalate
- Unsure of a command / need options → `?` first, then references, then source.
- Windows debugging → `references/windows.md`; remote gdb/windbg → `references/remote-access.md`.
- Full command/flag tables, ESIL opcode set, plugin dev, exact semantics → `references/` then `source/**/*.md` (authoritative, do not guess).

## References map
| Task | File |
|---|---|
| Mission workflows: CTF, malware, vuln research, lifter handoff | references/workflows.md |
| Decompile doctrine, YARA, Linux triage appendix | references/decompile-yara.md |
| CLI basics, flags, seeking, print, write | references/commands.md |
| Static analysis (functions, xrefs, types, vars, sigs) | references/analysis.md |
| Disassembly, print modes, ESIL/RzIL | references/disassembly.md |
| Maps, sections, open/map files, rebase | references/memory.md |
| Search engine | references/searching.md |
| Debugging, breakpoints, regs, stack, maps, threads | references/debugger.md |
| Windows debugging | references/windows.md |
| gdb/windbg remote, remoting | references/remote-access.md |
| Loops, macros, rz-pipe, JSON | references/scripting.md |
| Config/evars/themes/scripts | references/configuration.md |
| Math, $ variables, rz-ax | references/expressions.md |
| rz-bin (imports/exports/strings/etc) | references/binary-analysis.md |
| Other tools (rz-asm/hash/diff/find/gg/pm/run/sign) | references/tools.md |
