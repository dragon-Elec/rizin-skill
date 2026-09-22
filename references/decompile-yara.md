# Decompile + YARA + Linux Triage Appendix

Linux-only. No Windows bundle specifics. All commands one-shot, bash-safe.

## 1. Launch pattern (use this everywhere)

```bash
rizin -A -q -N -e scr.color=0 -c "<cmd>" -c "<cmd>" <file>
```

- `-A` runs full analysis (`aaa`) before your commands, so functions, xrefs, FLIRT names exist.
  Drop it (plain `rizin`) when you only want headers/strings and want to be fast.
- `-q` quits after `-c` (no interactive prompt to hang on).
- `-N` ignores user config so output is reproducible across machines.
- `-e scr.color=0` disables ANSI color — **important**, or output is full of escape
  codes that are hard to parse.
- Multiple `-c` run in order. Chain inside one with `;`.
- Read-only by default; add `-w` only to patch.
- Never execute untrusted samples or use `-d` without explicit authorization.
- Rizin can exit with code 0 after a bad command; inspect output for `ERROR:`
  and missing results — don't trust exit code alone.
- Keep output small: `~word` builtin grep (`afl~main`, `iI~bits,os`), `~?` counts
  matches (`ii~?`), `q`/`j` suffixes for quiet/JSON (`iiq`, `aflj`),
  `@ <addr>` for one command at a temporary offset (`pdz @ 0x401000`),
  bounded prints (`pd 40`), shell-side `head -n`.

**Reuse analysis instead of re-running it.** Each `rizin -A …` re-runs the expensive
`aaa`. For repeated queries, analyze once, save a project, reload without `-A`:

```bash
rizin -A -q -N -e scr.color=0 -c "Ps sample.rzdb" sample
rizin -p sample.rzdb -q -N -e scr.color=0 -c "afl" -c "pdz @ main"
```

## 2. Decompiler doctrine (pdz first)

**Default to `pdz` (rz-retdec) — reach for it first.** `pdg` (rz-ghidra) is the
better-known name so it's tempting by reflex — resist that: treat `pdg` as a
deliberate second choice (usually best structural recovery), and
`pdd` (jsdec) for a fast lightweight register-style pass. When one is unclear, run another.

| Command | Engine | When to use |
|---|---|---|
| `pdz` | rz-retdec | **Default.** Clean C, names imported APIs and FLIRT labels well. |
| `pdg` | rz-ghidra | 2nd choice — best structural recovery. Install via `rz-pm -i rz-ghidra`. |
| `pdd` | jsdec | Fast lightweight pass / quick look. |

- Add `o` for side-by-side offsets (`pdzo`, `pdgo`, `pddo`), `j` for JSON (`pdzj`).
- Decompile a specific function with `@`: `pdz @ fcn.00401000`.
- Gotchas: decompilers work on the *current function* — analyze first (`-A`/`aaa`,
  or `af` at the address) or you get *"No function at this offset"*. `pdg` is slow
  on very large functions, `pdz` is expensive on large binaries — decompile specific
  functions with `@ <addr>`, not the whole program.

**Cross-check loop (don't trust one decompiler in isolation):**
1. `afns` / `afx` — strings + refs a function makes; usually reveals its job.
2. `ii` + `axt @ sym.imp.<API>` — confirm which real API an ambiguous call resolves to.
3. `pdf` (raw disasm) — check what the decompiler glossed over.
4. `afn` rename once known (+ `afs` prototype, `afvt` local type) then re-run the
   decompiler so the name propagates and output sharpens.
5. Compare `pdz` vs `pdg` vs `pdd`; agreement raises confidence, verify critical behavior.

## 3. Triage / functions / xrefs upgrades

- `i` / `ia` — quick info / full summary. `iI` needs no analysis (run without `-A`).
- `iz~?` / `is~?` — string / symbol counts; cheap way to gauge a binary.
- `izz` scans the **whole** file (catches packed); `iz` is data-sections only.
- `aflt` — function **table**: size, xrefsTo, xrefsFrom, calls, basic blocks,
  cyclomatic complexity. Spot the big/central functions first.
- `afns` — strings referenced by the current function; one-line summary of its job.
- `pdsf` — function summary (strings, calls, jumps, refs) without full listing.
- `pxr` — hexdump with words annotated with refs (good for stack / GOT / PLT reads).

## 4. Analysis levels

| Command | What it does |
|---|---|
| `aaa` (= `rizin -A`) | Standard. Functions, calls, data refs, autonaming, **applies FLIRT**. |
| `aa` | Lighter: symbols + entry only. |
| `aaaa` | Experimental, aggressive. Try on stripped/obfuscated when `aaa` misses. |
| `aap` | Recover functions by scanning for prologues (calls don't reach them). |
| `Fl` / `Fa` / `Fs` / `Ff` | List / apply sigdb signatures, apply chosen FLIRT file, show current match. |

After `aaa`, FLIRT names static libs `flirt.*` instead of `fcn.*` — verify implausible
names/boundaries in disassembly. For huge cores/dumps: inspect `oml` first, constrain
with `e analysis.in=io.maps.x`, don't start with `-A`.

## 5. YARA (rz-libyara, Linux)

| Command | Purpose |
|---|---|
| `yaral <file.yar>` | Load `.yar`/`.yara`, apply rules, flag every match. |
| `yarad <folder>` | Same, recursively over a folder of rules. |
| `yaraM` / `yaraMj` | List all matches (plain / JSON). |
| `fs yara.match; fl` | Switch to `yara.match` flagspace, list match flags (then `s` to one). |

Match flags distinguish physical (`yara.match.pa.*`) from virtual (`yara.match.va.*`);
use matching `io.va=false`/`true` when reading bytes at a flag.

## 6. Linux format hints

- **ELF exe** — `iI` (arch/bits/PIE/NX/canary/stripped), then `ii` behavior,
  `iz`/`izz` strings. `s entry0` is startup; real logic is a few calls in —
  or `s main` if present.
- **Shared lib (.so)** — exports *are* the API surface: `iE`/`iEq`, decompile the
  interesting ones (`pdz @ sym.<export>`). `il` shows its own dependencies.
- **ELF core** — `oml` lists `PT_LOAD` maps; `ar`/`arj` saved registers
  (`ar PC`, `ar SP`). Rizin identifies the stack map on x86/x64/ARM/AArch64.
- **Raw / firmware** — supply assumptions explicitly:
  `-F any -a <arch> -b <bits> -m <base>`; modules/threads/discontiguous maps not inferred.
- **Dynamic resolution hides imports** — `dlopen` + `dlsym` patterns (packers/malware)
  hide real calls from `ii`. Spot via `afns` (resolved names appear as strings),
  confirm in disassembly.
- **Find dangerous capability fast** — `axt @ sym.imp.system`,
  `socket`/`connect`/`execve`, `mmap`/`mprotect`, `ptrace`, crypto APIs —
  jump straight to the code that uses what you care about.

## 7. Handy one-liners (Linux)

```bash
# Headers only, fast (no analysis)
rizin -q -N -e scr.color=0 -c iI ./sample

# Imports + strings overview
rizin -q -N -e scr.color=0 -c "iiq" -c "izzq" ./sample

# Analyze, then functions as a table
rizin -A -q -N -e scr.color=0 -c aflt ./sample

# Decompile one function (RetDec default; pdg / pdd alternatives)
rizin -A -q -N -e scr.color=0 -c "pdz @ main" ./sample

# Who calls a risky import?
rizin -A -q -N -e scr.color=0 -c "axt @ sym.imp.system" ./sample

# Export list for a .so
rizin -q -N -e scr.color=0 -c iEq ./libfoo.so

# Core-dump overview (no full analysis yet)
rizin -q -N -e scr.color=0 -c "iI" -c "omlj" -c "ilq" ./core
```

When a command's exact form is unclear, ask rizin: append `?` (`pdg?`, `i?`, `ax?`).

- `axt @@ii` — refs to **every import** at once (which functions touch which APIs).
- `axt @@f:str.*` — refs to every string (find code using a telling string).
- Loop: string/import → `axt` → function → `pdz` → `afn` real name → `afx`/`axt` outward.
