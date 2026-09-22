# Analysis — Static Code/Data Analysis

All under `a` family. `a?` for full list. Analysis output is also available from lib API / `rz-pipe` for custom loops.

## Analyze-all depth
- `aa` — basic: functions+basic blocks, symbols, entry. Stripped bins need deeper.
- `aaa` — adds calls/refs/strings/type matching. `aaaa` — + experimental (incl. ESIL emulation stage).
- CLI: `-A` runs `aaa`, `-AA` runs `aaaa`.
- Auto-analysis can be imperfect → use fine-grained commands below.

## Semi-automated program-wide passes
- `aab` — basic-block ("Nucleus") analysis.
- `aac` — analyze function calls (from selected/current fcn). `aaf` — all calls.
- `aar` — analyze data references. `aad` — pointers-to-pointers refs.
- `aaft` / `aft` — type inference. `aat` — propagate struct offsets (after linking types).

## Functions (`af`)
- `af [name]` — analyze function at current addr. `afr` — recursive.
- `af+ addr name [type]` — handcraft function (needs `afb+` for blocks). `af-` delete; `af-*` delete all.
- `afl` list functions (`aflj` json, `aflm` makefile, `afll` verbose, `afl=` ascii bars). `afi` function info.
- `afn name` — rename. `afm addr` merge. `afu addr` resize/analyze until addr. `aff` readjust.
- `afb` list basic blocks (format: start end jmp/fail sizes). `afb+` add block: `afb+ fcn blockaddr size [jump] [fail]`. `afB bits` set fcn bits (ARM/Thumb).
- `afc [cc]` set/get calling convention; `afcl` list; `afcr` register usage. Default cc via `analysis.cc` (user) / `analysis.syscc` (syscall). Loaded from `librz/analysis/d/cc-<arch>-<bits>.sdb`.
- `afo` fcn address; `afx` fcn refs; `afS` stack frame size; `aft` type-match fcn.
- Prelude for fcn detection: `e analysis.prelude=0x554889e5` (set BEFORE analysis).

## Variables (`afv`)
- `afvl` list vars/args. `afv- name|*` remove. `afva` re-analyze vars. `afvd` display value. `afvn new old` rename. `afvt name type` set type. `afvR`/`afvW` list read/write access sites. `afv=` with disasm refs.
- `afvr` register-based, `afvs` stack-based args/vars. `afv idx name type` define stack var at `[fp-idx]`.
- Auto var analysis on by default (disable `analysis.vars`). Depends on preloaded prototypes + calling convention (load symbols to improve). `afta` types analysis with vars.

## Cross-references (`ax`)
- `ax addr` add xref from current seek; `axc`/`axC` code/call, `axd` data, `axs` string. `ax-` delete.
- `axt addr` → xrefs TO addr. `axf` → xrefs FROM. `axtj`/`axfj` json. `axg` graph path to addr. `axtg`/`axg*` generate graph commands. `axt*` set flags on xrefs.
- Typical: see string in data → `axt` to find all references (data/code).

## Types (`t`)
- Basic types are definite-width (`int8_t`..`uint64_t`), NOT C-standard — `int`→`int32_t`/`int64_t` by platform.
- `t` list types. `td "struct foo {char* a; int b;}"` define. `to file.h` load header. `to -` editor. `dir.types` include path.
- `ts`/`tu`/`te` structs/unions/enums. `tp type [addr]` print struct at addr. `tpx type hex` print given bytes as struct. `tf` functions, `tt` typedefs, `tn` noreturn.
- Link a typed global to an address: `avga name type @ addr`; `avg` list globals. Then `aat` converts immediates to struct offsets in fcns.
- Structure member in disasm: `ahts offset` → list types with member at that offset; `aht ms1.member1` → set asm to show `[rsi + ms1.member1]`.
- Enum: `te name`, `te name val` → member, `teb name member` → value.
- Emulation inaccuracies: malloc return may differ between runs → set `ahr` return-value hint before `tl`/`aat`.
- `cf` (metadata format) is visual-only; `t` changes real analysis types.

## Signatures / FLIRT (`F`)
Supports HexRays FLIRT `.pat` (text) and `.sig` (compressed). Apply BEFORE matching analysis? — must analyze binary first.
- `Fc file` create sig; `Fd file` dump; `Fs file` apply; `Fa [filter]` apply from sigdb; `Fl`/`Flt` list sigdb.
- Config: `flirt.sigdb.path` (auto-apply if set before `aaa`), `flirt.ignore.unknown`, `flirt.node.optimize`, `flirt.sig.deflate/file/library/os/version`, `analysis.apply.signature`, `flirt.sigdb.load.*`.
- `rz-sign` CLI also creates/converts/dumps (see tools.md). Applied signatures stored in `flirt` flagspace.
- For `.a`/`.la` libs: unpack with `ar`, sign the `.o` files.

## Virtual tables (`av`)
- Check `analysis.cpp.abi` first. `av` list vtables; `avr` parse RTTI at vtable addr; `avra` search+parse all; `avrr` recover classes; `avrD name` demangle. `avg` globals. Basic support.

## Analysis configuration (common)
- `analysis.hasnext`, `analysis.jmp.after` — continue analysis past end/jumps. `analysis.jmp.indir` follow indirect jumps. `analysis.pushret` treat `push;ret` as jmp. `analysis.nopskip`. `analysis.fcn_max_size`.
- `analysis.noncode` — analyze data sections as code (malware/packed).
- Ref options: `analysis.jmp.ref`, `analysis.jmp.cref`, `analysis.datarefs`, `analysis.refstr`, `analysis.strings` (strings refs disabled by default — slower).
- Ranges: `analysis.limits`, `analysis.from`, `analysis.to`, `analysis.in` (io.maps, bin.sections, dbg.maps, dbg.stack, dbg.heap, analysis.fcn, analysis.bb, range...). See `e analysis.in=??`.
- Jump tables: `analysis.jmp.tbl` (experimental), rely on `analysis.jmp.indir` + `analysis.datarefs`.
- ARM/Thumb auto-detect uses partial ESIL (slower); override per-fcn with `afB`. MIPS: `analysis.gp`, `analysis.gpfixed`.
- Emulation in loop: `analysis.esil`; `emu.write` allows VM to modify memory (dangerous, needed for unpacking).

## Analysis hints (`ah`)
Override opcode/meta properties when analysis imperfect.
- `ahi base [n]` — immediate base (2/8/10/16/binary/octal/syscall/string/ip/port). `ahc addr` jump/call target. `ahd opcode` rewrite shown disasm. `ahs` size. `ahf` fallback. `ahS` syntax. `ahp` pointer. `ahr` return. `ahv` value. `aho` opcode type. `aht struct.member` struct offset. `aha`/`ahb` arch/bits. `ahl` list; `ah-*` clear.
- `ao` shows full opcode info (bytes, esil, jump, fail, type, cycles) — check before hinting.

## Gotchas
- Run `aa`/`aaa` (or `-A`) before `pdf`/xref/function queries.
- `aaa` may drop code that analysis deems unreachable (e.g. after unconditional `exit`); if missing, use `afr` from the fcn, or `af-*` + `aa` + targeted `afr` (see IOLI 0x07 example).
- Rizin may not auto-recognize jump tables; define manually (`Cd` data + `afb+` blocks) or hint.
- Concrete filter/search examples: `afl~sym.` (functions starting with `sym.`),
  `afl~main`, `/x 55` (`55` = `push rbp`, x86-64 function prologue).
