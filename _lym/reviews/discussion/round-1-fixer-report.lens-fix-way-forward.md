```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether every way forward, refusal, row and test the fix introduced can be reached and leads to progress. The fix introduced these:
- D2: the wait-only wrap on the row's other entries, the rule against a second fixer, the live-record `Expired` probe, and the finalize halt with the row's resume way forward.
- D4 step 1: a run attaches to or stops a live fixer from the key directory before its gate run.
- D4 step 3: the recorded tree identity is rewritten at the re-run.
- D4 Bound: the hand-off depends on the caller's role, which the front parses and writes into the run request.
- D5: Plan-Write re-checks with `lym loom validate-plan --lens-review`, and a missing lens review record is always left for the writer.
- D7: the Webster fork text depends on the `card` key's `fixer`.
- D2, D4, D5 and D7: the new acceptance bullets.

Findings:

1. MEDIUM — D2's new acceptance bullet ("on Darn-Write, a restart while the fixer holds uncommitted edits waits on the fixer and judges the re-run, never halting on the `clean` entry") and D2's wait-only wrap paragraph.
   Summary: the bullet can't hold when the waited-on fixer ends dead or done but leaves uncommitted edits. That run halts on `clean`, and the halt doesn't carry the fixer's reply or dead note.
   Evidence:
   - D1 and D2 step 3 allow a dead fixer to leave "any uncommitted change it left is in the worktree". The open-risk and assumption lines keep that partial work on purpose.
   - In the finalize evaluation, the wait-only wrap on `clean` waits, then runs `loomshed.NewCleanGate`. That gate fails on any dirty path (`internal/loomshed/cleangate.go:22-25`).
   - `evaluateGate` then marks `verify` not reached (`internal/shuttleengine/wait.go:1642-1646`). So the run halts gate-failed on `clean`, which is exactly what the bullet says never happens.
   - The resume way forward hands the writer `clean` findings that list the paths. They don't say a fixer left them, because only the fixer entry adds the reply or dead note, and that entry is never reached.
   - To fix: limit the bullet to a fixer that leaves the tree clean, or have the wait-only wrap add the waited fixer's reply or dead note to its own failing findings.

2. MEDIUM — D4 Bound: "the front parses the caller's role from its strand name … A run whose role is `gate-fixer` or webster's `verify-fixer` never hands off".
   Summary: reed numbers a role that is already held. A second live fixer in the same slug then has a role such as `gate-fixer-2`, which an exact role test misses. Its gate call hands off and spawns a fixer of its own.
   Evidence:
   - `internal/reedengine/strand.go:243` formats strand names with `agentname.NumberRole(role, held)`.
   - `agentname.MatchesRole(role, base)` (`internal/agentname/agentname.go:242`) exists to match numbered roles.
   - Several Webster forks of one batch can each have a card-gate fixer live at the same time, so numbered fixer roles do happen.
   - To fix: the Bound should test the role with `agentname.MatchesRole` against `gate-fixer` and `verify-fixer`, and so should its acceptance bullet ("a gate call from a fixer strand … never spawn a fixer").

Tool calls:
- One grep for `NumberRole(` callers and for webster files that read `gateslot.Config`. My context had neither: I needed to know whether strand roles get numbered, and whether the Webster fork's renderer can read the `card` key's `fixer` (it can, through `cardgate.go`).
