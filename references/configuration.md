# Configuration

## Evaluable variables (`e`)
- `e key` get, `e key=value` set, `e key=?` list valid values, `e key=??` + descriptions. Multiple: `e a=1 b=2`.
- `el` list all with descriptions; `el <namespace>` filter (e.g. `el asm.`). `el* <ns>` output as rizin `e` commands.
- `e-` reset all. `e! key` invert bool. `ee key` edit with editor. `er key` make read-only. `es` list spaces. `et` show type.
- Namespaces: `analysis.`, `asm.`, `bin.`, `cfg.`, `cmd.`, `dbg.`, `dir.`, `emu.`, `esil.`, `file.`, `flirt.`, `graph.`, `hex.`, `io.`, `pdb.`, `scr.`, `search.`, `stack.`, `log.`, `diff.`...
- CLI: `-e k=v`. Temporary per-command: `@e:k=v,k2=v2`.
- Visual config editor: `Ve` (or `e` in visual mode), arrow keys, `q` exit. Prompt: `e scr.prompt.popup=true` for autocompletion popup.

## Common vars (see `el` for full)
- Colors: `scr.color` 0=none,1=ansi(16),2=256,3=truecolor. `eco <theme>` pick theme; `ec` list colors; `ecs` palette; `ecr` randomize. `scr.randpal`.
- Display: `scr.utf8` (+`scr.utf8.curvy` curved corners), `scr.scrollbar` (flag zones; 1=right,2=top,3=bottom), `scr.visual.mode` (0 hex,1 disasm,2 debug,3 color blocks,4 strings), `scr.seek` (initial seek expr/flag like `eip`), `scr.wheel` (mouse in visual), `scr.gadgets`.
- Big-endian: `cfg.bigendian` (true=BE). Fortunes: `cfg.fortunes`, `cfg.fortunes.file` (tips/fun/custom path). `cfg.newtab` help on tab-completion.
- Prompt: `scr.prompt.vi` (vi mode), `scr.prompt.mode` (prompt color by mode). Dietline modes Emacs (default) / Vi.
- `cmd.repeat` — Enter re-runs last command.
- `stack.size` — stack hexdump size in visual debug.
- `dbg.follow.child`, `dbg.bep` (entry/main), `dbg.exe.path`, `dbg.funcarg`.
- Analysis/asm/search vars → their reference files.

## Colors & themes
- `scr.color` + `ec`/`ecs`/`ecr`/`eco`. Set in rizinrc to persist. Terminal env may limit to 0/1 (serial) vs 3 (modern).

## Initial scripts (startup)
Loaded during startup; disable with `-N` (no user settings/scripts) or `-NN` (no scripts/plugins).
1. **system** rizinrc (all users) — `rizin -hh` shows paths.
2. **user** rizinrc (`~/.rizinrc`, override with env `RZ_RCFILE`).
3. **binrc** — per-binary-format scripts in `~/.local/share/rizin/rc.d/bin-<format>/` (e.g. `bin-elf64/`, `bin-pe/`), executed for matching `i~format`.
4. **initial binary script** — `<binary>.rz` next to the binary; rizin prompts to run it.
- Scripts are interpreted as rizin commands. `RZ_RCFILE` env overrides rizinrc path.

## Run-time (environment) variables
`rizin -hh` lists them. Notable:
- `RZ_RCFILE` rizinrc path. `RZ_NOPLUGINS` don't load shared plugins. `RZ_DEBUG`/`RZ_DEBUG_TOOL` errors+crashes. `RZ_LOGLEVEL`/`RZ_LOGFILE`/`RZ_LOGCOLORS`/`RZ_LOGSHOWSOURCES`. `RZ_ABORTLEVEL`.
- `RZ_MAGICPATH`, `RZ_SIGDB`/`RZ_EXTRA_SIGDB`, `RZ_PIPE_IN`/`RZ_PIPE_OUT` (rzpipe fds). `RZ_PREFIX`/`RZ_INCDIR`/`RZ_LIBDIR`/`RZ_LIBEXT` paths. `RZ_USER_PLUGINS`/`RZ_LIB_PLUGINS`/`RZ_EXTRA_PLUGINS`.
- `RZ_CURL`, `RZ_DYLDCACHE_FILTER`, `RZ_HTTP_AUTHFILE`, `SFLIBPATH` (syscall libs), `DEBUGINFOD_URLS`, `ANSICON`, `COLUMNS`.
- `R2_GDB_PKTSZ` (gdb max packet size).

## Compile-time variables
`rizin -H` → build-time values (RZ_VERSION, RZ_PREFIX, RZ_SIGDB, RZ_USER_PLUGINS, RZ_IS_PORTABLE, ...). `rizin -H <var>` for one. See `source/configuration/compile_time_variables.md`.

## Gotchas
- `e k=??` (double ?) gives descriptions; `=?` only values.
- Settings persist via rizinrc; CLI `-e` overrides for the session without editing rizinrc.
- Some vars affect analysis/disasm retroactively (e.g. `analysis.prelude` must be set BEFORE analysis).
