---
verdict: APPROVED
---

The design matches the three scope points of board entry `batten-start-speed` and keeps to its Out list.

- Scope 1, measure and cut, every per-transition status push detached: D1 covers batten's seam, loom's and darn's seam and `lyx batten cleanup`; D2 measures from Worktree-Create to Run-Shed per row and inside each row, and cuts what it finds in batten's per-transition path; the acceptance criteria hold the entry's targets (a switched-off row and Open-Window each under 100 ms beyond the process).
- Scope 2, local-only pairs: D3 adds `lyx fabric add --local` with no origin probe and a remove with no push, archive tag or remote delete; the mark and the containment guard are what keep such a pair local and safe to remove.
- Scope 3, one `Command` producer: D4 replaces the three switched engines with one registry engine, gives each Go action a `lyx` verb, and settles the entry's two open questions (`run_settings` switches but cannot set a command, through `CategoryExecutable`; placeholders `{lyx}`, `{slug}`, `{task_dir}`).
  The separate switch key departs from the orch recommendation that an empty command is the off switch; the entry marks that recommendation as not a decision, and the switch key is what lets `run_settings` keep switching a row per task.
- Out: Weaver, `Job-Start`/`Job-Join`, async and `halt` modes, and moving `deploy`/`publish_verify` are all left out, as the entry says.

## Other board entries

- `row-entry-gate`: that note would move the switched producer's flag check into a shed-wide gate.
  The design builds no gate and keeps the switch inside `Command`'s config, naming the later move as an open risk, so it neither duplicates nor contradicts the note.
- `preflight-weft-push-sync`: Preflight's weft push is a row's own push, not a per-transition status push; the design leaves it to that note.
- `pair-from-parent`, `weaver`, `parallel-batch-parts`: `--local` is the local-pair primitive those notes need; creating a pair from a parent branch stays with `pair-from-parent`, and the design says so.
- `hub-workspaces`: a local pair gets no `_portals` link and no `_launchers` entry, so no hub workspace lists it; that note's future workspace lists an umbrella's worktrees, and whether a short-lived local batch pair counts as one is for that note to settle, with nothing here ruling it out.
- `regression-baseline-verify`: its `Job-Start`/`Job-Join` would extend `Command` later; the design keeps them out.
- `implementer-seam` depends on this task and takes nothing from its scope.
- `reed-status-buttons`: its configured commands may later share the placeholder expansion; the design builds none for it.

## Peers and parent commits

`lyx loom peers` lists no peer run and no parent commit (`own_row` 3), so there is no overlap and no landing order to give.
