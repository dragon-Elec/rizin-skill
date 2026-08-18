# Expressions & Variables

Rizin expressions = mathematical representations of values usable anywhere a number/addr is accepted. Support arithmetic + binary + boolean ops, flag names, seek positions, registers (debug/emulation).

## Evaluate with `%`
- `% expr` — print in all bases (hex/decimal/octal/binary/unit/string/float...). `% <expr>` for quick multi-format.
- `%vi expr` — integer only. `%v` — value. `%X expr` hex. `%b` binary, `%o` octal, `%u` units (K/M/G/T).
- `%r lo hi` random. `%h str` hash. `%f val bits` bitstring. `%l str` length (quiet → `$?`). `%b64`/`%b64-` base64. `%btw a b c` between.
- `%s start stop step` — generate number sequence (used with `@@` iterators).
- `%p vaddr`→paddr, `%P paddr`→vaddr. `%w addr` refs. `%i` input cmds. `%_` HUD.
- Conditional execution by `$?`: `%+ cmd` if >0, `%- cmd` if <0, `%! cmd` if 0, `%% cmd` if !=0.
- `%= expr` set `$?` without printing. `%== s1 s2` compare strings → `$?`.
- `%$ [var]` — list variables & values.

## Operators
`+ - * / % **` (power), `> <` (shift right/left), `~` (bitwise not), `| & ^`, `#` (rot left), `$` (rot right). Constrained to 64-bit ints / 64-bit floats.
- Quote to use `|` as OR not pipe: `% "1 | 2"`.

## Number bases / units
- `0x...` hex, `033` decimal-looking is decimal in `%` context, `sym.fo` resolves flag offset, `10K`/`10M` units.

## Usable variables (`%$?` / `$?` refcard)
Most useful:
```
$$  current virtual seek     $s  file size      $b  block size
$?  last comparison value    $S  section offset $SS section size
$B  base addr (lowest map)   $M  map addr       $MM map size
$FB function begin  $FE function end  $FS function size (linear)
$Fb basic block begin  $Fe basic block end  $Fs bb size  $FSS fcn size (sum bbs)
$j  jump target     $f  jump fail addr    $l  opcode length   $v  opcode immediate
$m  opcode mem ref  $w  word size (4/8 by asm.bits)  $c cols  $r rows
$p  getpid()  $P  pid of children (debug)  $O  cursor  $o  disk io offset
$D  current debug map base  $DB dbg.baddr  $DD debug map size
${ev}   value of eval var (e.g. ${asm.bits})
$r{reg} register value
$s{flag} size of flag   $e{flag} end of flag
$k{kv}  sdb query value
$Cn/$Dn/$Ja/$Xn  nth call/data-ref/jump/xref of function
flag   offset of flag (by name)
```
- Flags usable directly by name as offsets. `$e{flag}` = flag->offset+size.

## rz-ax (shell expression evaluator)
Same math outside rizin; multi-base calculator; interactive if no args.
- `rz-ax 1337`→`0x539`; `rz-ax 0x400000`→`4194304`; `rz-ax 3+0x80`→`0x83`; `rz-ax 0x80+3`→`131` (result base = first arg).
- `-s hex` hex→raw (`-S raw`→hex). `-b bin`→str, `-B str`→bin. `-e` swap endian. `-D/-E` base64. `-i` C byte array. `-n` int→hexpairs. `-o` octal→raw. `-I` IP<->long. `-t/-m/-W` timestamps. `-u` units. `-w size val` signed word. `-p` set-bit positions. `-L` bin→hex bignum. `-k` keep base. `-N` bin number. `-F` slurp file as hex.
- Full flag list: `rz-ax -h`. In-rizin equivalent is `%`.

## Gotchas
- `%` respects nothing about endianness for unit display; endianness affects byte-interpreted commands (`cfg.bigendian`).
- `%vi` gives plain int (for piping), `%` gives human multi-format.
- `$FS` = linear function size; `$FSS` = sum of basic-block sizes (can differ).
