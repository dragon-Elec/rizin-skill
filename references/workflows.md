# Mission Workflows — CTF, Malware, Vuln Research, Lifter Handoff

Task-oriented playbooks. Start one-shot (`rizin -A -q -c '...'`), go interactive only
when each step depends on the previous output (following xrefs, renaming as you go).

## CTF crackme ("find the flag / password")

1. `iI` — confirm arch/bits/OS.
2. `iz` + `izz~flag|pass|key` — interesting strings; note each string's address.
3. Works on stripped binaries where `sym.main` doesn't exist: take an interesting
   string's address and run `axt @ <addr>` to find the function that references it.
   That's almost always your check function.
4. `pdf @ fcn.<addr>` — disassemble. If it errors with
   *"Linear size differs too much from the bbsum"*, swap to `pdr` (recursive disasm
   that follows control flow). Same output, handles non-contiguous functions.
5. `pdc @ fcn.<addr>` for pseudo-C if available (`rz-ghidra` plugin via
   `rz-pm -i rz-ghidra`; if unavailable, fall back to `pdf`).
6. Look for `strcmp`, `memcmp`, crypto functions in imports (`ii`).

## Malware sample triage

1. `ii` — suspicious imports (`CreateRemoteThread`, `VirtualAlloc`, `WSAStartup`, etc.).
2. `iz` — C2 URLs, registry keys, mutex names.
3. `afl` — look for network, persistence, injection functions.
4. Rename and annotate as you identify each function's role (`afn name @ addr`,
   `CCa @ addr comment`, `f name @ addr`).

## Vulnerability research

1. `ii~(gets|strcpy|sprintf|recv)` — dangerous imports.
2. Find callers with `axt @ sym.imp.gets` (or whichever import).
3. Disassemble callers (`pdf`), check for bounds checking.
4. Look at stack frame sizes (`pdf` shows local variable offsets, `afS`).

## Lifter handoff (remill-lift / CFG export)

1. Open binary and analyze: `rizin binary`, then `aaa`.
2. Find function boundaries: `afl~main|target`, or `afl | grep` equivalent via `~`.
3. Disassemble: `pdf @ 0x401000`; control flow: `agf @ 0x401000`.
4. Extract address boundaries for lifter tools: `afi 0x401000` (shows size, basic blocks).
5. Call graph: `pdf @ main`, then `axf @ main` (what main calls), follow with `pdf @ 0x...`.
6. Indirect jumps complicate lifting (need symbolic execution):
   `afl` for candidates, then `/i "jmp.*r|call.*r"` or `/x ff25` (common indirect
   jump encoding), `pI 4096 @ 0x... | grep "call.*r"` for register calls.
7. Export: `agd @ 0x401000 > cfg.dot` then `dot -Tpng cfg.dot -o cfg.png`;
   `aflj | jq '.[] | {name, size, addr}'`; `rizin -A binary -c "pdf @ main" > main_disasm.txt`.

## Disassembly fallback + readability

- `pdf @ sym.main` → normal; `pdr` on bbsum error; `pdc` for pseudo-C; `pdg` graph mode.
- `e io.cache=true` (faster), `e asm.syntax=intel|att`, `e asm.comments=true`,
  `e asm.describe=true`. On large binaries use targeted `af @ addr` instead of full `aaa`.
- `pdg` requires `rz-ghidra`; install with `rz-pm -i rz-ghidra`.

## References

- Rizin docs: https://rizin.re/ — Radare2 book: https://book.rada.re/
- In-shell help: `?` at any time; `e asm.syntax=?` lists valid values.
