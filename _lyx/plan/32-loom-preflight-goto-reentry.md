# Card 32 — loom-preflight-goto-reentry

**Edit:**
- `internal/loomshed/loompreflight.go`
- `internal/loomshed/loompreflight_test.go`

**Uses:**
- `internal/loomengine/coherence.go`
- `internal/loomengine/report.go`
- `internal/shedengine/goto.go`

**Intent:**
Keep Loom-Preflight's half-finished stop coherent with PR #321 finding 1, which card 33 implements: goto will only move a run back, to a row at or before its current row.
Today the half-finished reason names `lyx loom goto --to Discussion-Write` as its way forward, a forward move from Loom-Preflight that card 33 refuses, so the named way forward would loop.

The half-finished check is reached only on a run moved back onto `Preflight` or `Loom-Preflight`: a fresh seed's history carries only the tolerated rows.
A run an operator deliberately moved back is not a fresh start the check protects, so treat that re-entry as a policy case and let it through:

- Loom-Preflight reads the status file's history (it is already told `statusPath` and `statusLockPath`).
  A re-entry is a history whose latest `goto` entry targets `Preflight` or `Loom-Preflight` and whose entries after that `goto` name only those two rows.
- On a re-entry, every `CheckHalfFinished` failure in the `CheckSeed` report is waived: it is logged with `logger.Warn` naming the failures and the re-entry, and the row returns `Done` when no other check failed.
  Any other failure still returns `Stuck` exactly as today.
- Without a re-entry, a half-finished report still returns `Stuck`, and its way forward becomes `way forward: seed a new run, or "lyx loom goto --to Loom-Preflight" accepts this run as a deliberate re-entry`.
  The `lyx loom goto --to Discussion-Write` text is removed.

Rewrite `loompreflight_test.go`'s goto-then-refuse test: after `goto --to Loom-Preflight` on a run whose history reaches `Discussion-Write`, `Call` returns `Done` with no reason, and the status file is unchanged by the call.
Add: the same history with no `goto` entry → `Stuck`, the reason names `seed a new run` and `lyx loom goto --to Loom-Preflight`, and carries no `--to Discussion-Write`;
a `goto` onto `Loom-Preflight` followed by a `Discussion-Write` entry → not a re-entry, `Stuck`;
a re-entry whose report also fails a non-half-finished check → `Stuck` naming that check.
Build the fixtures with `shedengine.Goto` and status files written as the verbs write them.

**ImpactSummary:** Loom-Preflight passes a deliberate goto re-entry and its half-finished way forward text changes; CheckSeed and other rows are untouched.
