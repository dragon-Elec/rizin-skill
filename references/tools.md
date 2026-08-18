# Companion Tools

All functionality also reachable from rizin shell / lib API. `rz-* -h` for exact flags.

## rz-asm (inline assembler/disassembler)
```
rz-asm [-ACdDehLBv w] [-a arch] [-b bits] [-m cpu] [-o addr] [-s syntax] [-f file] [-i skip] [-l len] 'code'|hex|-
-a arch   set architecture (-L list: columns a=asm,d=disasm,A=analyze,e=ESIL,I=RzIL)
-b bits   register size (8,16,32,64)
-d        disassemble hexpairs;  -D disasm with hexpairs
-e        big endian;  -E ESIL expression;  -I RzIL IL (validated)
-A        analysis info from hexpairs;  -w describe opcode
-k kernel  select OS/syscalls   -o/-@ addr   -s syntax(intel,att)
-c cpu    -f file    -F filt    -j json    -r rizin cmds    -x hex dwords
Env: RZ_ARCH/RZ_ASM_ARCH/RZ_ASM_BITS/RZ_BITS/RZ_DEBUG.
```
- Examples: `rz-asm -a x86 -b 32 'mov eax,33'` → `b821000000`; `rz-asm -a x86 -d 90` → `nop`. `echo 'push eax;nop' | rz-asm -f -`.
- Zero-pads with NOPs when `-l` > output.
- In-rizin: `wa`/`pa` assemble, `pd -1`/`pad` disassemble; `e asm.arch=??` same listing.

## rz-hash (checksums / encode / crypto)
```
rz-hash [-vhBkjLq] [-b S] [-a A] [-c H] [-E A] [-D A] [-s S] [-x S] [-f O] [-t O] [files|-]
-a algo   multiple: -a sha1,md4,md5,sha256; 'all' for every algo
-s string / -x hex / file / '-' stdin input
-b size -B per-block output    -f from -t to  -i times  -j JSON  -q quiet (-qq value only)
-c value  compare with expected    -e endian (big/little)
-E/-D enc/decrypt: -S key (-K hmac/key appending '^' pre, '@' file, '-' stdin), -I iv
-L        list algorithms (hash + crypto/enc): aes-ecb/cbc, blowfish, des-ecb, rc2/rc4/rc6,
          rot, xor, serpent-ecb, sm4-ecb, cps2, rol/ror + md2/4/5, sha*, sm3, blake3,
          crc8/16/32/64 family, adler32, fletcher*, xxhash, ssdeep, entropy, parity...
```
- Hash-with-context example: `rz-hash -qqE xor -K s:password -s hello` (XOR encrypt).
- In-rizin: `ph <algo>` current block (`ph md5`, `ph md5 32`), `phl` list algorithms.
- Note: full-file hashing buffers input in memory (don't hash huge files directly).

## rz-diff (binary diffing)
```
rz-diff [options] <file0> <file1>
-d algo   myers | leven | ssdeep (edit distance / similarity)
-H        visual hexdiff (side-by-side; keys: 1/2 pages, Z/A file0, C/D file1,
          G/B end/begin, N/M next/prev diff byte, /\/\/ 16-byte, <> 1-byte, : addr, ? help)
-S WxH    window size (min 120x20)   -t type  -j  -q  -a arch  -b bits
-t type   bytes|lines|functions|classes|command|entries|fields|graphs|imports|libraries|sections|strings|symbols
-0 cmd / -1 cmd  inputs for -t graphs (function name|offset) or command
-i        args instead of files (with -d)
-e k=v    eval var; -A vaddr/paddr; -B run aaa; -C no colors; -T timestamp; -v verbose
Colors: ec diff.unknown/match/unmatch in ~/.rizinrc
```
- Functions diff output columns: name0, size0, addr0, type (COMPLETE/PARTIAL/UNLIKE), similarity, addr1, size1, name1.
- `-t graphs -0 fcn -1 fcn` → graphviz/dot (DaruGrim-style bindiff) e.g. `rz-diff -g 0x40080d,0x40089f bin bin | xdot -`.
- In-rizin compare: `c` family (see commands.md).

## rz-find (search files from shell)
```
rz-find [-mXnzZhv] [-a align] [-b sz] [-f/t from/to] [-[e|s|w|S|I] str] [-x hex] -|file|dir..
-s str   string    -x hex    -e regex   -w wide   -I import   -S symbol
-z       zero-terminated strings   -Z show found strings   -X hexdump hits
-m       magic/file-type carver    -i identify filetype (magic db)
-E cmd   exec cmd per file found   -F file keyword    -M mask    -n batch
-r       rizin-command output      -j JSON    -q quiet
```
- Can replace `strings`/`file`: `rz-find -z`, `rz-find -i`.

## rz-gg (shellcode / tiny binaries)
- High-level lang (".r"/C via gcc/clang) → styled-for-injection x86/x86-64/ARM code.
- `-a` arch, `-b` bits, `-O`/`-F` tiny binary, `-f` format (C/PE/ELF/Mach-O/raw/python/JavaScript).
- Language: function sig `name@type(stack,static){body}`; types alias/data/inline/global/fastcall/syscall; `.var0`/`.arg0` stack-relative; `goto/while/if`; `:` inline asm; `write@syscall(4)` style. One-pass — define frame sizes/functions before use.
- Injection: raw eggs (position-independent). Shell `g?` has them. See `source/tools/rz-gg/*.md`.

## rz-run (controlled execution environment)
- Launcher: `rz-run [directives|script] [-- program args]`; also from rizin: `-r profile`/`-R directive`.
- Profile keys (key=value): `program`, `arg0..N`, `setenv=`/`unsetenv=`/`clearenv`/`envfile`, `stdin`/`stdout`/`stderr`/`input`/`stdio`, `timeout`/`timeoutsig`, `connect=host:port`/`listen=port`, `chdir`, `chroot`, `libpath`, `preload`/`rzpreload`, `setuid/_`euid/setgid`, `nice`, `aslr`, `pty`, `fork`, `bits`, `pid`.
- Uses: crackme env setup, fuzzing, redirecting stdio to socket/tty.

## rz-sign (FLIRT signature generation/conversion)
- `-o sigs.sdb lib` create SDB zignatures (analysis first). `-r` rizin cmds, `-j` json, `-a` more analysis depth.
- In-rizin: `zg` / `zos` (see analysis.md `F`). Exact usage: `source/tools/rz-sign/intro.md`.

## rz-pm (package manager)
- Install external plugins from sources: `rz-pm init` (clone db), `rz-pm -i <pkg>` (user) / `-gi` (system), `-u`, `-l`, `-s`, `-d`, `-c`, `-w`. e.g. `rz-pm -i lang-python3`, `rz-pm -i rz-ghidra`.
- Env: `RZPM_PLUGDIR`, `RZPM_BINDIR`, `RZPM_DBDIR`, `RZPM_GITDIR`, `SUDO`.

## Gotchas
- These tools are the CLI frontends; inside rizin prefer `%`(rz-ax), `ph`(rz-hash), `c`(rz-diff), `/`(rz-find), `wa/pd`(rz-asm).
- Never assume cross-tool flag uniformity; check `-h` per tool.
- rz-bin/rz-diff etc. are also usable in pipelines for automation (`rz-diff -t functions a b` for triage).