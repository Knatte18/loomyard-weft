---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1: `internal/runprofile/reinterpret.go` adds `ReinterpretArrasDefault` under the ID `profiles-arras-default-dropped`, registered on the profiles module in `internal/configreg/configreg.go`.
  It drops `defaults.arras` when the profile it names holds no arras block.
  `internal/runprofile/doc.go` records the rule.
  `TestReconcileHubWideAt_ProfilesStillLoadAfterTheArrasBlocksDrop` in `internal/configsync/configsync_test.go` pins the upgrade path through `ReconcileHubWideAt` and the registered reinterpretations.
  Commit `e6c2d8241`, then a follow-up commit from the self-check.
- F2: `_lym/landing/summary.md` widens the blast radius and says how a changed `defaults.arras` is handled.
  No commit, since the file is under `_lym`.
- F3: removed the duplicate `Default("arras")` assertion and its half of the doc comment in `internal/runprofile/runprofile_test.go`.
  Commit `0b3e76ed8`, with a `Loosened-Check` trailer.

A later commit rewraps the comments added for F1 in `reinterpret.go`, `doc.go` and `configsync_test.go` at sentence boundaries, as the gate's comment check asked.
It changes comments only.

Gate: tier0 over `configsync`, `runprofile` and `configreg` passed after the last change.
Tier 1 over `hubreconcile` passed after the F1 commit, with 9 targets skipped; it was not rerun after the later commits, which do not touch that package.

## Self-check

- fix-consistency, MEDIUM, summary bullet stale after F3 and missing the new test: fixed as proposed.
- fix-consistency, LOW, a dropped default reported in `rewrote` rather than `flagged`: fixed as proposed.
- fix-code-correctness, LOW, one walk mails the dropped default as dropped and as kept: fixed with our own change.
  The reinterpretation's line now says the drop holds even where another notice lists the default as kept.
  The follow notice itself is unchanged.
- fix-code-correctness, LOW, an alias or merge key makes the rewrite panic or falsely report a drop: fixed.
  Such a default is now flagged for a hand edit and the file is left as it is.
  No test row was added for this path, and no run exercised it.
- fix-affected-set, LOW, `lym config reconcile --layer default --apply` and `lym fabric reconcile` apply no reinterpretation: deferred.
- fix-proposal, LOW, the same finding as the line above: deferred.
- fix-proposal, LOW, the marker stops a second repair after an older binary adds the blocks back: deferred.
- fix-proposal, LOW (the "Related" note of the line above), a revert does not restore a dropped default: fixed.
  The summary's door line says so.
- fix-affected-set, LOW, second case of its finding (older binary, then marker): deferred, with the marker item above.

## Disputed

None.

## Deferred

- The two reconcile verbs that skip the walk.
  If one of them is the first to reconcile after the upgrade, it can commit a `profiles.yaml` that does not load, until a later walk repairs it, since the marker is still absent.
  Reason: `pattern/PATTERN-config-layers.md` lets only `hubreconcile`'s walk apply a reinterpretation.
  Changing which verbs run the walk, or moving the repair into the template follow, is a design choice beyond this round and applies to every reinterpretation.
- The once-per-hub marker.
  After it is written, an older binary can add the arras blocks back, the operator can set `defaults.arras` again, and the newer binary's walk then drops the blocks and keeps the default, with nothing to repair it.
  Reason: it needs a repair that is not a once-only reinterpretation, which is the same design choice.
  It needs an operator edit under an older binary.

## Rulings

None.
