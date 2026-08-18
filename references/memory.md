# Memory, IO, Maps, Sections

Everything in Rizin is treated as a file over an IO abstraction. IO plugins chosen by URI.

## IO URIs / plugins
`rizin -L` lists IO plugins. Notable:
```
file://,nocache://   local files
malloc://,hex://     memory buffer (rizin = == malloc://512)
dbg://,pidof://,waitfor://  native debugger
gdb://              gdbserver / qemu -s
winkd://            WinDbg KD (serial/network)
rap://,raps://      remote rizin protocol
rzpipe://, rzweb://
gzip://, zip://,apk://,ipa://,jar://
http://, tcp://, shm://, sparse://, null://, self://
```
Debug plugins have `d` in first column (`rwd`). `dL`/`Ld` switch debug plugin at runtime.

## Open / map files (`o`)
- `o file [addr [perm]]` open. `o+ file` open write. `ol` list opened. `o- fd` close. `o--` close all. `oc` relaunch. `oC len` malloc copy from current offset. `o=` ascii bars. `oL` list plugins.
- `oo[+bcdmn?]` reopen current file (variants: `oob` reload bin info, `ooc` as restart, `ood[fr]` reopen in debug, `oom` malloc, `oon` no bin info, `oonn` no bin info + header flags).
- `on[+]` open without parsing bin info. `ou fd` use fd. `op` prioritized file. `ox fd fdx` exchange descs.
- `oa arch bits [file]` specify arch/bits. `ob` handle binary files.
- Base address: CLI `-B addr` (PIE). For unknown-header files (bootloaders) map with `-m addr` / `o file addr` instead.
- `vfile://` = virtual file auto-created to patch relocations (avoids modifying original).

## Maps (`om`)
- `om fd vaddr [size] [paddr] [rwx] [name]` — create map. `oml` list maps. `omlj` json. `oml=` ascii. `om- id` delete map.
- Maps = what is actually mapped in memory (sections describe file contents; segments define mapping). For firmware that places sections at different addresses, use `iS` info + maps.

## Sections / segments (`i`)
- `iS` sections (paddr,size,vaddr,vsize,align,perm,name,type,flags). `iSj` json. `iS=` ascii bars. `iSS` segments.
- `i` bin info, `iI` full info (arch/os/bits/class/canary/PIE/NX...). `iE` exports, `ii` imports, `ie` entry, `ir` relocs, `iz`/`izz` strings.

## Virtual vs physical
- VA mode (default for executables): seek by virtual address; starts at entry. `-n` opens non-VA (file offset).
- `%P paddr` → vaddr; `%p vaddr` → paddr. `rz-bin -p` shows physical addresses.

## Reading memory
- `px`/`pxw`/`pxq`/`p8`/`pd`/`ps` at any address via `@`.
- In debug mode, process memory treated as plain file; all mapped pages readable. `dm.` shows current map. Registers usable as seek (`s rsp+0x40` in debug/emulation).

## Rebasing / loading libraries
- `-B` rebase base address. Map libraries to reproduce a core-file/debug environment with `o` + `om`.
- Reloc targets auto-patched into `vfile://` maps (see `ol` output for `reloc-targets`/`patched`).

## Gotchas
- Opening with `-w` needed to write. Writing only affects the map/file you target.
- Patching relocations uses `vfile` to keep original file/IO ranges unmodified.
- For 8051-style multi-address-space targets, pseudo-registers (`_code`, `_idata`, `_sfr`, `_xdata`, `_pdata`) hold base addresses; memory re-allocated on `aei`; see `source/arch/8051.md`.
