# Study: how much work, and how real, a WSL2 gate farm on Septim is in its stored and ephemeral modes

`septim-farm-study` · Task · research, gate, process

A desk study for note `remote-gate-farm`, which holds the design and the operator's bounds; read that note first.
The report lands under `docs/research/gate/`.

## Question

How much work is it, and how realistic, to run lym gate tier runs on Septim, a Windows 11 desktop (Ryzen 7 9800X3D) with WSL2 installed, on a closed LAN, while the hub and its agents stay on Neve (Linux, KDE)?
Answer it for each of the two modes:

1. **Stored:** the hub's code is checked out on Septim, with a warm build cache on WSL2's disk (private code such as loomyard).
2. **Ephemeral:** the code may not be checked out or stored on Septim; it is sent into a RAM disk for one tier run and deleted after (work code on OSL-1046).

## What the report covers

- The Septim side: WSL2 networking (mirrored mode or port forward), SSH into WSL2, start at Windows boot, systemd, Defender exclusions, sleep and Wake-on-LAN, the Go toolchain and tmux for tier2, and a .NET SDK for a C# hub.
- The lym side, read from this repository's gate code (the slot pool, the tier runner, `prebuilt_lym`, the detached gate runs, the go-json counter): what must change to send a tier run to a remote slot and stream its results back, and how a dead or asleep remote is found and fallen back from.
- For ephemeral mode: what stays on Septim during and after a run (tmpfs, build cache, swap, logs, test temp dirs, core dumps), and what it costs in speed without a warm disk cache.
- A rough size of the lym change in tasks, and the risks that could make either mode not worth it.
- A recommendation for each mode.

## Bounds

- A desk study only: it reads this repository and public documentation. It runs no gate tier, no benchmark and no command on Septim, and it reaches no machine but Neve.
- No web search beyond what WSL2, OpenSSH and tmpfs behaviour need; at most 15 pages read.
