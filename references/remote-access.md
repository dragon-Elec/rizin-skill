# Remote Access & Debug Backends

## gdbserver (gdb remote protocol)
```
rizin -d gdb://<host>:<port>          # connect (debug plugin auto-selected)
rizin -D gdb gdb://<host>:<port>      # explicit plugin
rizin -d gdb://<host>:<port>/<pid>    # attach to pid (extended mode)
[0x...]> doof gdb://<host>:<port>/<pid>  # start debugging after analysis (rebases session)
```
- Rizin needs the binary locally to load symbols. If remote isn't literal `localhost` (or file absent), it prompts to download; manual: `R! download_file <remote> <local>`.
- Symbol/base fixes: `-e dbg.exe.path=<local>`, `-e bin.baddr=<baddr>`.
- Packet size: env `R2_GDB_PKTSZ` (e.g. `=512`), or runtime `R! pktsz` / `R! pktsz N`.
- `R! ?` lists gdb IO commands: `R! pid`, `R! pkt s`, `R! rd` (reverse-debug availability), `R! dsb`/`R! dcb` (step/continue backwards — only on gdbservers like Mozilla rr), `R! monitor cmd` (hex-encode monitor cmd), `R! detach [pid]`, `R! inv.reg`, `R! pktsz`, `R! exec_file`, `R! download_file`.
- Debug protocol logging: `R! monitor set remote-debug 1`, `R! monitor set debug 1`.
- **Locality note:** only literal `localhost` triggers a local binary search; `127.0.0.1` is treated as remote.
- Use standard rizin debug commands after connecting.

## Rizin's own gdbserver
```
rizin =
[0x...]> Rg <port> <file> [args]   # start gdbserver (Rg! = protocol debug msgs)
```
Then connect with any gdb-compatible client: `rizin -d gdb://localhost:<port>`.

## Remoting (rap:// and R commands)
Everything uses the IO subsystem → rizin can run as a server controlled remotely.
- Server: `rizin rap://:1234` (host1), connect from another `rizin rap://:1234` (host2).
- Client `R`:
```
R+ rap://<host1>:1234//bin/ls        # add host (also dbg:///... inside URI)
R                                   # list connections (fd numbers)
R 0 px / R s 0x666                  # run command on host fd
R= <fd>                             # interactive session on host ('q' quit)
R- [fd]                             # remove hosts/close
R! <cmd>                            # run via rz_io_system
R< <fd> <cmd>                       # send output of cmd to remote fd
Rt <[host:]port> [cmd]              # start tcp server
Rh[*?]                              # HTTP webserver commands
Rg[!]                               # start gdbserver
R=!  /  R!= <fd>                    # enable/disable remote cmd mode
```
- Redirect output to a TCP/UDP server: `R+ tcp://host:port/`, then `R<5 cmd`.
- Full example topology (two rap hosts + localhost client) → `source/remote_access/remoting_capabilities.md`.

## WinDbg KD (kernel) — see windows.md
`winkd://` serial/network kernel debugging; `-D winkd`.

## Generic IO/debug notes
- `rizin -D <backend> <uri>` forces a debug plugin; switch at runtime with `dL`/`Ld`.
- `R!` invokes the active IO plugin's `system()`; most provide `R! help`.
- Debugging over SSH on macOS may block on auth prompt (see `source/debugger/apple.md`).

## Gotchas
- Local-file existence is decided by the literal hostname `localhost`, not loopback IPs.
- Remote reverse debugging requires backend support (non-default gdbservers).
- Don't assume GDB/WinDbg semantics map 1:1 to Rizin; check `R!`/`d?` per backend.
