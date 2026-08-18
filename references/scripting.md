# Scripting & Automation

## Command composition
- `;` — sequence commands. `|` — pipe rizin output to EXTERNAL command (`ao|grep address`).
- `` `cmd` `` — command substitution (backticks): output used as argument. Example: `px 10 @ `ao~ptr[1]``.
- `>` / `>>` — redirect to file. `%$?` gives helpful variables (`$v` immediate, `$m` mem ref).

## Loops / iterators (`@@`)
- `@@f:regex` — run command over matching flags (`afi @@f:fcn.* ~name`).
- `@@F` — over every function. `@@i` — every instruction in current basic block; `@@i @@b` — all bbs of current fcn.
- `@@=o1 o2 ...` — over a list of offsets (`ao @@=$$ $$+2`). `@@.file` — offsets from file (one per line).
- `(_;cmd;?e)() @@b` — run a sub-command per bb with blank-line separation.
- `@@?` — full list.

## Macros (`(`)
- `(name; cmd1; cmd2)` — define. `.(name)` — call. `(name arg0 arg1; ...)` — with args, reference `${arg}`.
- `..(name a b c d)` — call multiple times with successive args.
- `(*` — list; `(-name` — remove. Useful for automation inside a session.

## Aliases (`$`)
- `$alias=cmd` — alias a command (e.g. `$disas=pdf`). `$alias` → run. `$alias=` undefine.
- `$alias=$some text` — alias that prints text. `$` — list all aliases. `$alias?` — show aliased cmd.
- Aliases can contain aliases (`$pmore='b 300;px'`).

## Initial/script files
- `-i file` — run script AFTER file load; `-I file` — BEFORE file load.
- `. script.rz` — interpret a script inside session. `#!lang` — rlang scripts.
- `rizinrc`, `binrc` (`rc.d/bin-<format>/`), `${bin}.rz` auto-load (see configuration.md).
- `rizin -c 'cmds' -q` — one-shot batch. `-q` quiet + quit after `-i`/`-c`; `-qq` force quit.

## rz-pipe (IPC — scriptable from external languages)
Interact with a rizin instance: spawn pipes, http, tcp. `pip install rzpipe`.
```python
import rzpipe
rz = rzpipe.open("/bin/ls")
rz.cmd('aa')
print(rz.cmd("afl"))      # raw output
print(rz.cmdj("aflj"))    # parse JSON -> object
```
- Python supports pipe/spawn/http/tcp/rap/json. Haskell/OCaml: pipe/spawn/http/json. Rust: +async. Ruby: RzPipe.
- HTTP server: `rizin -qc=h /bin/ls` then `rzpipe.open("http://127.0.0.1:9090")`.
- See `source/scripting/rz-pipe.md` for Haskell/OCaml/Rust/Ruby snippets.

## Machine-readable output (agent-friendly)
- Many commands accept a `j` suffix for JSON: `aflj`, `iIj`, `iSj`, `pxj`, `pfj`, `olj`, `dmj`(?), `axtj`, `dbtj`, `dmi` tables, etc. **Verify per command** — not universal.
- Pretty-print JSON: append `~{}` → `olj~{}`.
- Internal grep for columns: `~[n]`, rows `~:n`; combine `pd 20~call:0[0]`. `~{` json indentation, `~{path}` json path.
- Table query syntax for `aflt`/`is`/`izz`/`il`-style outputs: `cmd:col/sort:...:output` (sort, cols, gt/ge/lt/le/eq/ne, uniq, page/head/tail, str, strlen/minlen/maxlen, sum; outputs `csv/json/fancy/simple/quiet`). See `source/tools/rz-bin/tables.md` + `:?` in-session.
- `rz-bin -r` / `-Ir` / `-Sr` / `-zr` / `-sr` — emit rizin-command scripts for automation (see binary-analysis.md).

## Agent workflows
- Batch extraction: `rizin -qc 'cmd' file > out.json` / `-qc 'iij'`.
- Snapshot state: `dr* > regs.saved` (save), `drp regs.saved` (restore).
- Automate per-hit: `e cmd.hit="cmd"` in searches; per-breakpoint: `dbc 'cmd' @ addr`.
- Chain: search → `@@f:hit*` → apply command to every hit.

## Gotchas
- `|` pipes only to external programs (not internal). Use `;` for internal chaining.
- Backtick substitution runs before arg parsing — mind quoting in nested cases.
- `j` JSON availability must be confirmed per command; don't assume.
