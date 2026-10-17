---
verdict: APPROVED
---

## Measure

The board entry `gate-wait-cache-cap` with its operator decision of 2026-10-17.
The decision record has no `Added after Discussion` entry, so the board body is the measure unamended.
The record's one open risk, a caller parsing the old `ok: false` busy envelope, names no unrecorded parent correction, and no review segment carried another risk over.

## Diff against the entry

PR 715 at `27491d046` against `main`, which it has fully merged in:

- Shared cap from one definition: `gatecli.Command` passes `shuttleengine.DefaultAwaitCap` as the slot wait, and `cmd/testtiming` takes the same constant; no copy exists.
- Busy answer: with every slot held at the cap, the verb prints an ok envelope with `slot_free: false`, the holders by worktree and site, and a run-again message, and exits 75, which the help, the doc and the stencils call no failure.
- Gate rule: `internal/shellguard/rows.go` adds "run the same command again in a turn that does nothing else, so the prompt cache stays warm between calls", the wording `lyx shuttle await`'s help uses (`internal/shuttlecli/await.go:40`); the claudeengine golden settings change only by that text.
- `cli_wait_sec`: removed from `Config`, `Limits` and the template, and listed in `RetiredKeys`, so a hub file that still sets it loads and reconcile prunes it; a test pins the load.
- Out-of-scope points stay untouched: no change bounds the test run after the slot is taken, and the hub's `cli_wait_sec: 180` is left for the orch.
- Docs: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, the `DefaultAwaitCap` comment, `internal/shuttleengine/doc.go` and the gate line of `docs/overview.md`.

Incidental: commit `e19f3078c` drops a 20 ms lock-grace override from the `internal/shedverbs/loopwait_test.go` fixture to stop a flake under full-suite load.
It is test-only, changes no behaviour and does not reach into another entry's scope.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commit outside this branch, and `own_row` is 22.

- `landing-trailer-text` (row 17, Merge-In): both change `docs/overview.md` and `internal/shuttleengine/doc.go`, at different places, the landing module line and engine-ledger prose against the gate line and the `AwaitAny` paragraph.
  Text-only overlap, and its decisions on landing trailers and the engine ledger do not touch the gate's slot wait or `DefaultAwaitCap`.
  Landing order: this run (row 22) lands first, and `landing-trailer-text` takes these changes at Merge-In.
- `creel-cli-output-fixes` (row 12, Webster): no shared path or behaviour.

## Superseded

No board entry is made obsolete; `gate-language-neutral` extends the gate pool to other languages and stays open.
