# Plan-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed; nothing is deferred.
`lyx loom validate-plan` passes after the edits (`ok: true`, informational `create-new-unit` lines only).
The overview's `approved:` key is untouched.

## Fixed

- **F1 (MEDIUM), card 1:** the Intent now says how an `AttachGated` run reaches a turn boundary: it starts mid-turn, and the Done its replay of `events.jsonl` reports is its first gated arrival, where restart re-evaluation and a waiting `notify` are reached.
  It names the residual: a writer mid-turn at the restart looks idle after replay, so one carried prompt can land in that turn.
  It also gives the bound: the closure's on-disk schedule.
  `gate_pending_test.go` gains an `AttachGated` case, and `Uses` gains `Runner.AttachGated`.
- **F2 (MEDIUM), card 1 and overview:** a non-nil `Final` on a non-`PassOnCap` entry is a gate error at evaluation, with its own test; the Bound sentence and the overview's gate-contract Shared Decision say `PassOnCap` only for both pending and `Final`.
- **F3 (MEDIUM), card 2:** `parent_review_wait_min` below 1 is refused in its own check, not the sibling timeout loop, with a refusal naming `attempts: 0` as the off switch; `config_test.go` covers 0 and a negative value.
- **F4 (LOW), card 1:** a pending outcome is not memoised past its arrival; the test list gains a verdict recorded between poll ticks being read.
- **F5 (LOW), card 5:** `gate_test.go` gains the cap and throttle across a restart and a notify recorded between two closures.
- **F6 (LOW), card 7:** the `docs/overview.md` edit also adds `parent_review_wait_min` to the loom row's key list, a sentence on the status waiting note, and an `internal/parentreview/` tree line.
- **F7 (NIT), card 12:** a Bound line covering the gate entry and the widened discussion commit pathspec.
- **F8 (NIT), card 7:** `approve --review` resolves against the invoking cwd, as `reject`'s file does.

## Deferred

None.
