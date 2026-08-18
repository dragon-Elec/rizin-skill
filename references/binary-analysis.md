# Binary Analysis — rz-bin (static inspection)

`rz-bin` extracts info from executables (ELF, PE, Mach-O, Java CLASS, + plugin formats). Presents in formats other tools (incl. rizin) accept.

## Usage
```
rz-bin [-AcdeEghHiIjlLMqrRsSUvVxzZ] [-@ at] [-a arch] [-b bits] [-B addr] [-C F:C:D]
       [-f str] [-m addr] [-n str] [-N m:M] [-P pdb] [-o str] [-O str] [-k query]
       [-D lang symname] file
```

## Key options (static workflow)
```
-I     binary info (arch,bits,class,endian,os,canary,PIE,NX,relro,stripped...)   <- identify
-i     imports (+ PLT offsets)      -E  exports      -e  entrypoint
-l     linked libraries (direct deps only)   -L  list bin plugins
-s     symbols      -S  sections      -SS  segments      -SSS  sections->segments
-z     strings from data section     -zz  raw strings (whole file)     -zzz  dump raw
-r     output as rizin commands      -j  JSON      -q  quiet (-qq fewer fields)
-K algo  checksums (e.g. -K md5 -S)  -O  write/extract ops      -x  extract bins
-D lang name  demangle      -P/-PP  PDB info / download      -@ addr  section/sym/import at addr
-g     show all info     -H  header fields     -T  file signature     -U  resources
-m addr  source line at addr     -M  main symbol addr     -Y  base candidates (firmware)
```
- `-f mach` select sub-binary; `-F binfmt` force plugin; `-B addr` override base (PIE); `-a arch`/`-b bits`.

## Rizin-format output (feed into rizin)
- `rz-bin -Ir file` → `e` commands (cfg.bigendian, asm.bits/arch/os, bin.lang...).
- `rz-bin -Sr` → flags each section start/end + rest as comment (`fs sections; f section..text ...`).
- `rz-bin -zr` → flag strings + `Cs` (mark as string), prepopulated `strings` flagspace.
- `rz-bin -sr` → flag symbols + define ranges as functions/data.
- Pipe into rizin: `rizin -qc '. <(rz-bin -r file)'`-style, or use `-n`/`-r` combos. (These scripts are the bridge between rz-bin and in-session analysis.)

## Operations (`-O`)
```
rz-bin -O h      # help: d/s/1024 dump symbols; d/S/.text dump section; c codesign; C entitlements
rz-bin e/0x8041111 somefile.bin   # change entrypoint
rz-bin p/.bss/rwx somefile.bin    # set .bss perms rwx
rz-bin d/S/.text somefile.bin     # dump .text as hex stream
```

## Strings
- `-z` uses data sections (rodata/.text); `-zz` scans raw. Example HTB: "Wrong Password!" only found by `-zz`.
- `-N min:max` force string length range; `-n str` find named section/symbol/import; `-@ addr` at address.

## Notes
- `-l` lists only DIRECT library dependencies (does not follow transitive deps like `ldd`).
- `rz-bin` in-session equivalents: `iI`, `ii`, `iE`, `iS`, `iz/izz`, `is`, `ir`, `ie`.
- For symbol/struct/class details and exact tables → `source/tools/rz-bin/*.md`.

## Gotchas
- `-z` vs `-zz` differ (section-scoped vs whole-file) — use `-zz` when `-z` misses.
- JSON is via `-j`; rizin-format via `-r` (not JSON).
- `rz-bin -P` needs PDB config (`pdb.server` etc.) for `-PP` download.
