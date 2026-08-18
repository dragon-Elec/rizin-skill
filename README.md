# rizin

An agent-oriented knowledge base for [Rizin](https://rizin.re), the free and open-source reverse-engineering framework. It turns the full Rizin Book into a three-layer hierarchy an AI agent can work through: a tiny always-loaded operational memory, task-focused references, and the original documentation as the authoritative fallback.

## How to install

Install via [Smithery](https://smithery.ai), the cross-runtime skill registry. Replace `<agent>` with your runtime's slug (e.g. `kilo`):

```sh
smithery skill add https://github.com/dev1912-sbt/rizin-skill --agent <agent>
```

For the list of supported agents and their slug values, see the [Smithery managing-skills reference](https://github.com/mikekelly/managing-skills/blob/main/SKILL.md).

The folder name **must be** `rizin` — that is the skill's `name` and how the loader matches it.

After install, the skill triggers on phrases like *"analyze this binary"*, *"debug this executable"*, *"disassemble this function"*, *"find the password in this crackme"*, *"inspect ELF/PE imports and strings"*, etc. (see `SKILL.md` for the full description).

## Layout

```
rizin/
├── SKILL.md          # Operational memory — small, loads fast, enough for normal sessions
├── references/       # 13 task-oriented operational references
│   ├── debugger.md          # debugging, breakpoints, regs, stack, maps, threads
│   ├── analysis.md          # static analysis: functions, xrefs, types, variables, signatures
│   ├── disassembly.md       # print modes, asm.* config, ESIL/RzIL
│   ├── memory.md            # IO, maps, sections, open/rebase
│   ├── searching.md         # search engine
│   ├── windows.md           # Windows debugging (messages, DbgEng, KD)
│   ├── remote-access.md     # gdbserver, remoting, WinDbg
│   ├── scripting.md         # loops, macros, rz-pipe, JSON/machine output
│   ├── commands.md
│   ├── configuration.md
│   ├── expressions.md
│   ├── binary-analysis.md   # rz-bin
│   └── tools.md             # rz-asm, rz-hash, rz-diff, rz-find, rz-gg, rz-pm, rz-run, rz-sign
└── source/           # The original Rizin Book, untouched — complete and authoritative
```

## How to use it

1. `SKILL.md` first — enough to start an ordinary analysis or debugging session.
2. Escalate to `references/` when the task needs specialized depth.
3. Escalate to `source/` for exact semantics, obscure commands, or anything undocumented in the condensed layers.

Nothing in the condensed layers is assumed to be exhaustive; `source/` is the source of truth.

## License & Attribution

`source/` (and the condensed layers derived from it) comes from the [Rizin Book](https://book.rizin.re) ([github.com/rizinorg/book](https://github.com/rizinorg/book)), which is licensed under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) — see [`LICENSE`](LICENSE).

The original book files are included unmodified; the `SKILL.md` and `references/` layers are adapted material derived from it. Under CC BY 4.0 you must keep this attribution and license notice when redistributing the content.