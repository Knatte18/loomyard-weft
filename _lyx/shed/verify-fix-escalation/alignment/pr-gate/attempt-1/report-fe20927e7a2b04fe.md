---
verdict: APPROVED
---

## Alignment at PR-Gate — verify-fix-escalation

Measured: the diff of pull request 855 at `7e529e10` against `main` (44 files), against the board entry `verify-fix-escalation`.
The decision record has no `Added after Discussion` entries, and the run has no parent, so the board body is the measure unamended.

### Scope

The entry asks that a verify failure the fixer fork does not clear escalate in model inside Webster's gate, before the run halts:
the last attempt, or the attempt after a fork that made no commit, goes to a cold fixer at the recovery role, spawned as its own strand the way `recover-batch` spawns one, with the verify-gate report as its brief.
The operator view adds one Sonnet attempt, then the Opus-high fixer.

The diff does that and no more:

- `internal/websterengine/verifygate.go` routes each failure to `fork` or `cold`: cold at the cap, after a fixer that added no commit, and from then on.
- `lyx webster fix-verify` (`internal/webstercli/fixverify.go`, `internal/websterengine/verifyfixer.go`) spawns the cold strand at the `recovery` model-spec with the verify-gate report as its brief, blocks like `recover-batch`, and records the spawn in `State.VerifyFixers`.
- `template.yaml` drops `verify_gate_attempts` to 2, which gives one fork attempt, then one cold attempt.
- The Master template, the verify-fix stencil split, the reclaims, the write evidence, `doc.go`, `docs/overview.md` and the specs follow from the new fixer.

Two changes sit outside the decision record's own scope list but serve the entry:
the `batchpause.go` refactor lets `fix-verify` refuse under a pause as the batch verbs do,
and `recover-batch` now stops a strand it cannot record, the same guard `fix-verify` got in review.
The latter breaches the record's "batch recovery path out of scope" line, which is the writer's own decision and not a scope deviation;
it is three lines of strand-leak hardening on the spawn path the entry names as the model.

### Open risks carried over

Darn-Review round 2 carried two findings, `reclaim-leaves-cold-fixer-record-nonterminal` and `shared-fixer-body-claims-unapplied-run-exit-audit`.
Commits `f1bc24e7` and `62bb9db8` address them.
Neither names an unrecorded parent correction, so neither escalates.
The record's own two open risks, routing going cold one attempt early and a cold fixer lacking batch history, stay inside the entry's design.

### Overlaps

- **Peer `loom-tail-review`** (row 7, Plan-Write; this run's `own_row` 21), with no changed paths yet.
  Its decision record lists Webster's own gate and its fixer, naming this entry, as out of its scope, and keeps the `round` gate kind because Webster's verify-gate fixer runs its call.
  It replaces the Webster-Review perch with a Code-Perch, which this run's docs name as a later reader of the fixer commits; that is text the peer rewrites at its landing, not a conflicting design.
  Landing order: this run is further along (21 against 7) and lands first; `loom-tail-review` takes its changes at Merge-In.
- **Parent commits**: none listed.

### Board entries touched

- `weaver` (note) states that Webster's verify-fix forks run on Master's model; after this run that line describes the fork path only.
  Weaver's own goal, a cold implementer per batch with a model ladder, stays open, so it is not superseded.
- `fixers-on-sonnet` concerns the Plan and Discussion fixers and the `webster_fixer_model` perch key, not the verify gate; no overlap.

No board entry is made obsolete by this run.
