---
verdict: APPROVED
---

The design matches the board entry `shuttle-open-permission`.
The entry asks for a `--permission-mode` flag (bypass or prompt) or a config default on `lyx shuttle open`, so an operator session opens in bypass under its lyx strand name.
D1 delivers the flag with the values `bypass` and `prompt` and declines the config default, which the entry offers as an alternative, not a requirement.
The strand name is `lyx shuttle open`'s existing behaviour, which the design keeps.
D2 to D5 carry the mode through `OpenSpec` to the Claude engine's launch and resume lines and add nothing the entry rules out.
The entry's precondition holds: `creel-stage1-fixes` has landed as `27887027b`, the branch's base.

## Board fit

The other entries that touch shuttle (`codex-engine`, `creel-agents`, `rename-binary-lyd`, `spawn-audit-log`, `session-sync`, `pause-background-work`) neither cover the open permission mode nor contradict the design.
`codex-engine` plans its own approval and sandbox modes for a Codex engine;
D2 leaves each `SessionOpener` engine to honour or refuse the mode, so a future Codex engine maps it itself, with no conflict.

## Overlap

`lyx loom peers` lists no peer runs and no parent commits (`own_row` 3), so there is no overlap and no landing order to give.
