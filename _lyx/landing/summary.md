# webster: recover from re-begun batches, card rewrites and regressions

A webster run now recovers on its own from three states that used to need a hand edit or a human.

## What changes

- **Re-begun batch (#329).**
  A batch that was begun but never recorded (its fork landed nothing) no longer makes `begin-batch` refuse a later `Uses:` of its Create targets or Rename New sides.
  Those targets count as forthcoming, as an unbegun card's do.
  `begin-batch`, `run` entry and `lyx webster validate` now scope dispatch validation by one shared rule, so the pre-flight verb answers what the run will answer.
- **Webster's own rewrites (#330).**
  Handle canonicalization, handle binding and exact-tier drift repair update the recorded `CardHashes` of each begun card they changed, in the same restamp that adopts the rewrite.
  `rebaseline --card NN` on a pending card no longer fails on begun cards webster itself rewrote.
- **Fork self-fix (#332a).**
  The fork implementer prompt treats a failing test the card's change caused as that card's gate failure.
  The fork fixes it under the existing self-fix cap and reports `FAILED` when the cap runs out.
- **Integration regression (#332b).**
  When the run-exit integration stage triages a regression, it spawns one cold strand at the recovery role (`recovery_timeout_min` timeout) before escalating.
  The strand's prompt comes from the new stencil `webster-template-integration-fix`.
  Go re-runs the plan's `## verify:` and triages again.
  With no regression left, the outcome stays done and `summary.md` and the integration report record the fix commits and the identities they cleared.
  Otherwise the run escalates as before, recording the pre-fix bisect result under the `-1` record.

`CLAUDE.md`'s TEMPORARY items for these three bugs are deleted.
The `internal/websterengine` package doc and `contracts/specs/webster-spec.md` describe the new scoping and the integration-fix stage.
`contracts/specs/refusal-spec.md` gets a row for the unfixed-regression refusal.

Closes #329
Closes #330
Closes #332

## Design decisions

- **Forthcoming targets are a bound on #329.**
  A `Uses:` of a target from a begun, non-terminal card passes validation unresolved.
  The producing card's done-checks at `record-batch`/`recover-batch` still fail its batch when the target is missing, and a batch that never reaches a terminal record keeps the run from an outcome of done.
  Terminal batches' targets do not count as forthcoming.
- **`validate` stops resolving begun, non-terminal cards.**
  Drift in such a card's own targets is no longer reported by any of the three sites.
  Its bytes stay pinned by its recorded `CardHashes`, and its done-checks and drift detection still run at record time.
- **Hash moves only at sanctioned restamps.**
  The update runs only where the foreign-edit check already passed, never from `rebaseline`.
  It moves a card only when its recorded hash equals the pre-rewrite `PlanFileHashes` entry for its file.
  A foreign edit to a begun card is still refused by `rebaseline`, `record-batch` and recovery.
- **One fix attempt per run, no new config key.**
  The attempt is recorded in `state.json` when spawned, so a resumed run never spends a second one; `run --fresh` resets it.
  The fix strand's GUID is recorded too, so `run`'s entry-time reclaim stops it.
- **Strand commits are checked before triage.**
  Go accepts them only when the commits from the pre-fix head to the report's `head_sha` are all non-merge, none touches the plan directory or `_lyx`, and HEAD reconciles against `head_sha`.
  Otherwise the attempt fails.
- **Go never resets the branch on a failed attempt.**
  The strand's commits stay, and `summary.md` and the escalation's way-forward name the pre-fix head and those commits for the operator to decide.
- **The post-fix triage decides success, not the strand's report.**
  Triage sees only failures, so a test the strand deleted or skipped would read as cleared.
  The prompt forbids deleting, skipping and weakening tests; the later review rows see the fix commits like any other.
- **Out of scope.**
  What a card verifies and re-running integration tests after a rework fork belong to `verify-publish-hardening`.
  `state.json` of runs begun before this lands is not repaired: a run already carrying stale `CardHashes` still hits the #330 refusal.

## To check by hand

- The integration-fix strand has run only against a fake starter in tests; no real tmux/reed strand has run it.
- Run `update-plugins.sh` (operator only) to deploy the new and changed stencils; `lyx` warns that the board copies of `landing-template-describe` and `webster-body-implementer` have drifted from the worktree source.
- The final integration run at HEAD exited 0 with `status: OK`; no `-tags integration` suite was reported as run beyond the plan's `## verify:`.
- `lyx webster status` lists a stale `-1` record for batch 7's fix-commit-check card as `stuck` while batch 7 itself is `done`; check that it is harmless.
