# Plan round 1 — fixer report

All 14 findings were checked against the code and the decision record, and every premise held.
Every finding is fixed.
`lyx loom validate-plan` exits 0 after the fixes; it reports only informational `create-new-unit` findings.

## Fixed

- **F1 (card 20):** added `contracts/specs/refusal-spec.md` to the targets.
  The Intent names the two new loom rows, "plan-write archive: webster busy" (transient) and "plan-write archive collision" (correctness halt), each with its way forward, and the seam-error test step asserts it.
- **F2 (card 7):** the five recorded states are now rows of the existing `TestBeginBatch_Refusals`: its two report rows are reworded and four are added, replacing the planned new table test.
- **F3 (card 11):** each behavior is tested once, at the surface that owns it.
  The `--batch` pairing goes in untagged `reset_test.go`.
  Resolutions and target refusals become steps of `TestPlanReset` in `reset_integration_test.go`.
  HEAD, files, `mutations`, the `target`/`sha` envelope keys and the held-lock acceptance or refusal become steps of `TestResetCmd`.
- **F4 (card 14):** the Intent now names the three call sites of `fingerprintMismatchWayForward`.
  `PlanEditError` and `batchCardEditError` pass the plain run, and `Run` passes `deps.reentryStep()`.
  `PlanEditError` keeps its signature, so its four callers need no edit, and the ImpactSummary states that.
  `BeginBatch` is dropped from the targets, since nothing in it changes.
  The two functions are covered by the card's existing `internal/websterengine/fingerprint.go` file target.
  Listing them as member glyphs tripped `containment-file-overlap` against card 19's file target.
- **F5 (cards 2, 9):** added `tools/tokencount#StartFit.write` to both cards.
  Card 2 makes it read `Orientation` so the tree builds.
  Card 9 rewords its printed label.
- **F6 (card 12):** standalone `PlanReset` skips the foreign-dirty-path refusal, through a `ResetDeps` flag set only when there is no fabric, because keep is the guard (D4's bound).
  The Intent states what that lets through and what bounds it, and that pairs keep the refusal.
  The standalone integration steps cover a foreign dirty file kept and keep's refusal over run-written and foreign paths.
- **F7 (cards 11, 6, 16):** each card now names the existing refusal-spec rows it rewrites or deletes.
  Card 11 rewrites `reset: unknown --to` and `reset: run busy`.
  Card 6 rewrites `batcher.yaml did not load` into the config-invalid row.
  Card 16 deletes `rebaseline: begun card of an unfinished batch` and trims the matching clause of `rebaseline card set changed`.
- **F8 (card 4):** the `TestReconcileAll` rows now cover only the wiring: the dry run reports and writes nothing, apply writes a loadable file, and an absent file is seeded, not migrated.
  The rewrite-content assertions stay in card 3's table.
- **F9 (card 8):** the CLI's `executionBatches` computes the base only when `st` is nil and otherwise passes the zero `StartBase`.
- **F10 (cards 10, 12):** card 10 no longer creates the `resetKeep` wrapper.
  Moving it to card 12 was blocked by two gate rules.
  A `plan:` handle declared in card 12 would have no later reference (`handle-unreferenced`).
  A `gitwrap.go` file target in card 12 overlaps card 13's member glyphs there (`containment-file-overlap`).
  So card 12's standalone function in `reset.go` calls `gitrepo.Repo.ResetKeep` directly, with no one-use helper.
  `reset.go`'s header names that call as webster's one mutating git operation.
- **F11 (card 4):** added `internal/configcli/doc.go` and `internal/configcli#Command` (the reconcile long help).
  Both now state the migration, the dry-run listing and the `migrated` key.
- **F12 (card 6):** the pre-run test gains a malformed and an unreadable `webster.yaml` step, asserting `config_invalid` present and absent, with `state.json` unchanged.
- **F13 (card 5):** added `internal/shedrecipe/entries_simple_test.go`, with `AnchorPath` in the `LoomPreflight` row's `validatedFields`.
- **F14 (card 12):** "runs no git" now reads "runs no mutating git", noting that the uncommitted listing is a read, and the bound says it moves no commit or file of the tree.

## Disputed

None.

## Deferred

None.
