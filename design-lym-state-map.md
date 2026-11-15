# Map every piece of state lym keeps, where it lives and whether git holds it

`lym-state-map` · Task · research, batten, tender, dobby, orch, fabric, loom · running · Record-Write

## Goal

Map every piece of state lym keeps for a hub and its runs, as the groundwork for the Motif that redesigns adopt.
This is a read-only study: it reads code and the records on this machine, runs no adopt, starts no run and changes no code.

## What the report holds

- Every piece of state lym keeps: what it is, the file or place it lives in (`.lym/` in the prime, Bolt, each pair and its weft, `~/.claude`, tmux, creel, `/tmp`), and the code that writes and reads it.
- For each: whether git holds it, on which branch or repository, and whether it is pushed when it changes.
- For each that git does not hold: what breaks when another machine lacks it, and whether lym could rebuild it from pushed state.
- The steps `lym adopt` takes today (since the 2026-11-11 rework, commits `0deeb9b9` to `90d11895`),
  each with the state it reads and writes, and which of them can fail when the old owner's machine is off or holds unpushed work.
- The adopt failures on record: the adopt records under `_bolt/.lym/adopt/`, the board notes the Motif bundles, and the logs they name.
- The states a run passes through today, as batten, loom, dobby and tender record them, as a first draft of one FSM.

## Bounds

The report lands under `docs/research/` as a commit.
It names facts and sources; the design is the Motif's.
