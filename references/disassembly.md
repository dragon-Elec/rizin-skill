# Disassembly & Intermediate Languages

Disassembly is a print mode under `p`. `rz-asm -L` or `e asm.arch=??` list arch plugins.

## Commands
- `pd N` — disassemble N instructions (negative = backward). `pD N` — disassemble N bytes. `pdj`/`pDq` json.
- `pdf` — disassemble current/function. `pdg` — Ghidra decompile (`rz-ghidra`, install via `rz-pm -i rz-ghidra`; like IDA F5). `pdJ` json w/ text.
- `pda` — disassemble all opcodes (byte-per-byte; useful for ROP). `pdb` basic block. `pde` follow execution flow. `pdl` w/ sizes. `pdp` follow pointers (ROP chains). `pdr` recursive across graph. `pdR` recursive block without analysis. `pds` summarize. `pdk` all methods of class.
- `pi`/`pI` — instruction-only (simpler output). `pa`/`paD` assemble/disasm in print.

## Print-mode details
- `pd N @ addr`, `pdf @ sym.main` — temporary seek via `@`.
- Block default limit; set `b` or use `@!N` for more.
- `ao` — full opcode info (bytes, size, esil, jump, fail, type, cycles, stackop). `aoi`/`aoip` — RzIL of N instructions.

## Disassembler config (`asm.*`, 130+ vars — see `el asm.`)
- `asm.arch`, `asm.bits` — architecture/word size. `asm.cpu` (e.g. AVR model). `asm.os` — OS/syscall table.
- `asm.bytes` — show raw bytes. `asm.offset` — show/hide offsets. `asm.syntax` — intel/att/masm. `asm.pseudo` — pseudocode output.
- `asm.lines.call` / `asm.lines.out` — control-flow lines. `asm.fcn.size` — fcn size in header. `asm.tabs`/`asm.tabs.once` — operand spacing.
- `asm.sub.jmp` — show jump targets as function names. `asm.sub.reg` — replace regs with arg names (e.g. A0/A1). `asm.sub.rel` — show PC-relative refs as string refs. `asm.sub.section` — prefix offsets w/ section name. `asm.sub.varonly` — var names in disasm.
- `asm.describe` — opcode description comments. `asm.cmt.*` — comment options. `asm.flags`, `asm.trace`, `asm.calls`.
- `asm.ucase` — uppercase. `e asm.bits=?`/`e asm.syntax=?` list valid.

## Pseudo / syntax examples
- `asm.pseudo=true`: `r2 = (0x1e << 16)`, `[r1 - 0x80] = r1`, `goto 0x101048`.
- `asm.syntax=att`: `movq (%rsi), %rbx`. `masm`: `mov rbx, qword [rsi]`.

## ESIL (Evaluable Strings Intermediate Language)
Forth-like, comma-separated, stack-based semantics of each instruction. Enable: `e asm.esil=true` (visual `O` toggles). Used for emulation and `asm.emu` comments.
- View: `e asm.emu=true` — computed reg/mem values as comments (+ `likely` for predicted jumps). `e emu.str=true` — compact useful info (strings, jump likelihood).
- Example: `push ebp` → `8,rsp,-=,rbp,rsp,=[8]`; `mov eax,[0x80480]` → `0x80480,[],eax,=`.
- Ops: `==` compare, `< <= > >=`, `<< >> <<< >>>` shifts/rotates, `& | ^`, `+ - * / %`, `!` neg, `++/--`, `= += -= *= /= %=`, `=[]` poke, `[]` peek, `?{...}` conditional, `$` syscall, `TRAP`, `SWAP/PICK/RPICK/DUP/NUM/CLEAR/BREAK/GOTO`, `TODO` (unimplemented). Whitespace stripped.
- ESIL flags prefixed `$`: `$z` zero, `$c` carry, `$o` overflow, `$p` parity, `$s` sign, `$b` borrow, `$r` regsize, `$ds` delay slot, `$jt/$js` jump target. `$0,of,=` reset a flag.
- Non-assoc order: `a,b,-` = b−a; `a,b,/=` = b/=a.
- Emulation: `aei` init VM, `aeim` init memory(stack), `aeip` set IP, `aer` regs; `aes` step, `aeso` step-over, `aesu addr` until, `aesue expr`, `aec` continue. Cannot emulate external calls/syscalls/SIMD → emulate small chunks (crypto/unpack).
- `asm.esil` shows ESIL; check `ao` output; `analysis.esil` uses in analysis loop; `emu.write` allows writes.
- Full opcode table + REP handling + x86 detail → `source/disassembling/esil.md`.

## RzIL (new IL, BAP Core Theory-like, LISP s-exprs)
- `aoi` one-line RzIL, `aoip` pretty, `aoj` json, `plf` whole function inline.
- `rz-asm -Ie -a ppc <hex>` → RzIL; `-E` → ESIL.
- Emulation via `aez*`: `aezi` init VM at offset, `aezs` step, `aezse` step+show changes, `aezsu` until addr, `aezv` print/modify VM vars. Requires `e io.buffers=true` for memory ops.
- Arch supports RzIL if `I` in `La` listing (e.g. `ppc`).
- More: `source/disassembling/rzil.md`.

## Metadata in disassembly (`C`)
- `CC "comment"` add comment (visual `;`). `CC,` / `,` — link a text file comment.
- `Cd N` define data; `Cs N` define string; `Cf N fmt` define format struct (visual-only). `C-*` clear.
- `Cv` comments on vars/args. `CS` meta spaces. `Ch` hidden, `Cm` magic.
- Import external annotations: `. file.rz` (e.g. from `rz-ida.py` IDB converter).

## Gotchas
- `pD` takes byte count; `pd` takes instruction count — don't mix.
- Function view (`pdf`) requires analysis; else `p: Cannot find function`.
- ROP gadgets search: `/R` (searching.md), `pdp` for chains, visual `pg` gadgets are unrelated to ROP gadgets.
