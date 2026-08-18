# Searching

Search engine initiated by `/`. Everything is a file (socket, process mem, device — all searchable). Hits auto-flagged for later reference.

## Commands
```
/ string       search plain string (\xHH escapes allowed)
/i str         case-insensitive
/w str         wide string (f\0o\0o\0); /wi wide ignoring case
/x hex         hex bytes; /x a1..c3 ignore nibbles (mask ff00ff); /x a1b2:fff3 bitmask
/b[/x/...]     search backwards (add b to any: /bx, /bw)
//            repeat last search
/e /RE/i       regular expression
/a asm        assemble instruction, search its bytes
/ad/ mnem     search assembly category/pattern, e.g. /ad/ jmp qword [rdx]
/c asm        search asm code
/F file       search content of a file (from/offset/size)
/p size       pattern search (repeated sequences, no explicit pattern needed)
/P size       similar blocks
/R            ROP gadgets search/list/query
/r addr       analyze opcode reference to offset
/v[248] num   search asm.bigendian value
/z min max    search strings of given size
/m            magic/file-type search (carve)
/Ca           search expanded AES keys
/c            cryptographic material search
/s            entropy search (flags sections as entropy_section_N)
/d hex        deltified byte sequence
/o[/O]        offset of N instructions backward
/g            graph paths A to B
/! ff         first occurrence NOT matching
```
- Results become flags `hit0_0`, `hit0_1`, ... in the `searches` flagspace. `ps @ hit0_0` to read. Remove with `f- hit*`.
- `cmd.hit` — command run on each hit (automation). Multiple: separate `;` or a script `. script`.

## Configuration (`e search.` — see `ell search`)
- `search.in` — boundaries: `io.maps`, `io.map`, `bin.sections`, `bin.section`, `bin.sections.rwx`, `.r`, `.rw`, `.rx`, `.wx`, `.x`, `dbg.stack`, `dbg.heap`, `dbg.map(s)[.x/.r/.rw...]`, `analysis.fcn`, `analysis.bb`, `raw`, `block`. Restrict for speed: `/ @e:search.in=io.maps.x`.
- `search.from`/`search.to` — range (inclusive/exclusive). `search.align`. `search.chunk`. `search.distance`. `search.flags` — flag hits (vs print only). `search.maxhits`. `search.overlap`. `search.prefix` (default `hit`). `search.show`. `search.case_sensitive` (smart/sensitive/insensitive).
- Ctrl-C interrupts; current pos flagged `search_stop`.
- `search.esilcombo` — stop after N consecutive hits.
- `search.kwidx` — last search index.

## AES keys
- `/Ca` finds **expanded** AES keys (not plaintext). Searches to `search.distance` or EOF.
- Plaintext AES: `is~AES` (symbol). High entropy may hide secrets: `/s` entropy scan (set `b` ~4096 first), flag sections `./s*`, loop `px 32 @@f:entropy*`.
- No absolute detection — understand the binary.

## Backwards
`/b` searches before current offset. `s <addr>p` seeks to a physical/previous; to re-find earlier results seek back then `/b` again. Each `/b`/`/bx` re-runs from current position backward.

## Syscall/asm search (analysis-adjacent)
- `/as` (with ESIL stack `aei`/`aeim`) — find syscalls, list by name. `/ad/` for assembly search. `/ad/ svc` on ARM.
- `asl` — list supported syscalls for platform. `asr N` syscall num→name, `asn name` name→num. `as?` more.

## Gotchas
- Large searches (esp. to stdout, `/R`, magic) can be slow — restrict range with `search.in`/`search.from`/`search.to`.
- `/*` starts a multiline comment, NOT a search (close with `*/`).
- Search hits are re-flagged on each run; clean with `f- hit*`.
