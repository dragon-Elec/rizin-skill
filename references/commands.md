# Commands — CLI Fundamentals

Rizin CLI: `[.][times][cmd][~grep][@[@iter]addr!size][|>pipe] ; ...`

Rizin is not a shell. Don't type `afl | grep foo` or `afl > file.txt` expecting shell
semantics. Use rizin's own operators: `~foo` grep-style filter (e.g. `afl~main`),
`~{}` JSON pretty-print, `> file` redirect (works), `| shellcmd` pipes to shell
but quoting is fragile. `;` chains internal commands (`s 0x401000; pdf 10` — not `&&`).
Many commands need `@` for address: `pdf @ 0x401000` (not `pdf 0x401000`).

- `!cmd` → shell command. `!!cmd` → shell, stdout back to rizin.
- `| program` → pipe command output to external program. `|H` html, `|.` = alias.
- `> file` / `>> file` → redirect output to file; `2>` stderr; `H>` html.
- `;` → run multiple commands on one line.
- `3px` → repeat command N times.
- `~` → internal grep (line filter). Useful when no external grep/Windows. `~[col]`, `~:line`, `~keyword`, `~!` negate, `~?` count. See `~?`.
- `_` → reprint last command output.
- `R! cmd` → pass to current IO plugin (debugger backend). `R! ?` for backend help.
- `%` math evaluator; `` `cmd` `` command substitution (backticks) as arg; `$alias` aliases; `(macro` macros; `@@` iterators.

## Temporary modifiers (`@`)
Applied left-to-right, restore afterward.
```
@ addr            temporary seek
@!blocksize       temporary block size
@e:k=v,k2=v2      temporary eval vars
@a:arch[:bits]    temp arch/bits
@b:bits           temp asm.bits
@f:file           replace block with file contents
@s:string         replace block with string
@x:hexstring      replace block with hex
@r:reg            seek to register value
@i:nth.op         seek to Nth relative instruction
@F:flagspace      temp flagspace
@k:key            seek at sdb key value
@o:fd             temp fd
@B:nth            nth instruction in current basic block
@@=o1 o2 ...      foreach over list of offsets
```

## Seeking
- `s addr` → seek (accepts math expr: flags, `+ - * /`, `[mem]`, regs in debug).
- `sd delta` → relative seek. `s++`/`s--` → block-sized seek.
- `shu`/`shr` → undo/redo seek history (`sh` list, `sh*` rizin cmds). In visual: `u`/`U`.
- `sg`/`sG` → begin/end of section/file. `sf [fcn]` → next/specific function; `sf.` → current fcn start.
- `so N` → seek N opcodes. `sr reg` → seek to register.
- `sa align` → seek aligned. `s.` honor core offset.
- Current address: `s` alone; `spad`. `$$` = current seek in expressions.
- `%` eval: `% 0x100+200` → multi-format; `%vi expr` → int; `%v` → value.

## Block size (`b`)
Determines how many bytes commands process when no size given.
- `b N` set, `b+ N`/`b- N` adjust, `bf flag` → size of flag, `bm N` max.
- `pD @ $FB !$FS` → disassemble current function (function begin/size vars). `pdf` doesn't use/affect global block.

## Flags & flagspaces
Flags = named bookmarks with offset+size. Grouped into flagspaces.
- `f name @ addr` add; `f-name` remove; `fl` list; `fr old new` rename; `f.` local flags (scoped to function, names like `loop`); `fz name` flag zones (scrollbar); `fC` comment; `fc` color.
- `fs name` select flagspace; `fsl` list; `fs *` all; `fs- name` remove; `fsm` move flags at addr; `fss` stack.
- Common flagspaces: `functions`, `imports`, `sections`, `strings`, `symbols`, `searches`, `syscalls`, `relocs`.
- `@@f:regex` → run command over matching flags (see scripting.md).

## Printing (`p`)
- `px` hexdump, `pxw` 32-bit words, `pxq` 64-bit words, `p8 N` hexpairs, `pxo` octal.
- `pd N` disassemble N instructions (negative = backward); `pD N` disassemble N bytes; `pdf` function; `pdj` json; `pdq` quick; `pdp` pointer/ROP chains; `pds` summarize; `pdr` recursive.
- `pf fmt` formatted data (structs) — see `pf??` formats, `pf???` examples. e.g. `pf xxS @ rsp` stack args; arrays: `pf 2*xw a b`.
- `ps` string (auto), `psz` zero-terminated, `psw`/`psm` UTF-16 LE/BE, `psW`/`psM` UTF-32, `psp` pascal, `psb` all strings in block.
- `pc` C array, `pcp` python, `pcg` golang, `pcj` json, `pcy` yara, etc. (see `pc?`).
- `pt` timestamps (`ptd` MS-DOS, `pth` HFS, `ptn` NTFS), `pt.` now; `cfg.datefmt` strftime.
- `pi`/`pI` instruction-only output. `pv` variable/pointer view. `po` op on block. `ph algo` hash block. `ppd N` de-bruijn pattern; `ppd/ val` find offset (respects `cfg.bigendian`).
- `p6e`/`p6d` base64. `pr` raw bytes (dump to file: `pr N @ addr > out.bin`).
- JSON pretty: append `~{}` (e.g. `olj~{}`). Grep columns: `~[0]`.

## Writing (`w`) — requires `-w` or `oo+`
- `wx hex`, `w "string"`, `wv value`, `w0 N` zeros, `wa "asm"`, `wz` zero-terminated, `ww` wide.
- `wo` ops on block: `wox val` XOR, `woa` ADD, `wos` SUB, `wom` MUL, `woA`/`woo`/`wo2/4/8` endian swap, `woE/woD` encrypt/decrypt block.
- `we` extend/insert bytes; `wc` write cache; `wu` unified patch; `wf` from file/socket.
- `wD N` de-bruijn write; `wD/ val` offset (like `ppd/`).
- `r N` resize file (neg truncates).

## Comparing (`c`)
- `cx hex` compare bytes at current offset (sets `$?`). `cc addr` compare block to addr; `cd addr` diff disassembly; `cf file` compare to file; `c1`/`ca`/`cb` variants. See `c?`.

## Yank/Paste (`y`)
- `y N` yank N bytes to clipboard; `yy` paste; `yt N addr` yank-to (copy); `ys`/`yp`/`yx` show clipboard as string/raw/hex; `yf` from file; `ywx` from hexpairs.
- Visual: cursor select (`Vc`, SHIFT+hjkl) then `y`/`Y`.

## Gotchas
- JSON output is not universal — verify per command (append `j` only where supported; `?` or `~` grep the help to confirm).
- `~` (internal grep) is preferred over spawning external `grep` (portable, works on Windows/embedded).
- Offsets in commands can be math expressions with flag names, `$$`, registers, memory derefs `[addr]`.
