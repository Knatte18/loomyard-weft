# Read the Septim gate farm study and decide each mode

`septim-farm-followup` · Note · research, gate

Depends on: [`septim-farm-study`](design-septim-farm-study.md)

When the Septim study lands, its report under `docs/research/gate/` answers how much work, and how real, a WSL2 gate farm on Septim is in its stored and ephemeral modes.

The operator reads it with the orch and decides, for each mode, whether to build it.
A mode chosen to build goes into note `remote-gate-farm`, which then runs as a motif; a mode not chosen is dropped from that note.
