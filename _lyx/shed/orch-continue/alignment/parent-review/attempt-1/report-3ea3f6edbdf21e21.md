---
verdict: APPROVED
---

## Scope against the board entry

The entry `orch-continue` asks that `lyx orch` record its session id and offer `lyx orch start --continue`, or make continue the default for a dead strand, so a reboot brings back the same orch conversation without digging the id out of `~/.claude/projects/`.
The entry carries no separate Done-when; its body is the goal.

The design meets it:

- D4 records the provider session id in orch state at every launch and keeps it current across `/clear` cycles through the watcher.
- D1 and D2 add `--continue` as an explicit flag that takes the existing adopt path with the recorded id; the entry offers the flag or the default as alternatives, and D1 picks the flag.
- D3's guards, D6's status line and D1's envelope hint serve the entry's stated pain, an operator who loses the conversation after a reboot, and add no work another entry owns.

The Out bullets leave reed's rebirth replay to reed and transcript copying to `session-sync`, so the design takes no work from another entry.

## Other board entries

- `creel-notices` (note): moves the watcher's notices, injects and idle probe to creel and keeps the watcher's state and cycle logic; D4's id tracking sits in that kept state, so there is no contradiction.
- `orch-split` and `orch-any-worktree` (notes): neither is built or in flight; the design adds no prime-bound or single-orch assumption beyond the state directory orch already has.
- `session-sync` (note): explicitly out of scope.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits and one peer.

- `creel-stage1-fixes` (row 7, Plan-Write): both change `internal/shuttleengine`.
  This run adds a session-id field to shuttle's read-only session reading (D5) and returns the minted id from the start seam (D4); the peer changes input-box captures, the idle probe's stall reasons and a creel interrupt variant.
  Different behaviour at different places, and no decision or PATTERN entry of one touches the other's: a text-only overlap.
  Landing order: `creel-stage1-fixes` at row 7 is further along than this run's `own_row` 3, so it lands first and `orch-continue` takes its changes at Merge-In.

No shared bookkeeping file is named yet; `cmd/lyx/testdata/test-budget.yaml` may become one at Plan.
