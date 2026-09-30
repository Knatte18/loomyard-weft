# Fixer report — discussion round 1

## Fixed

- **F1 (MEDIUM), confinement note and open risks.**
  Removed the "change is confined to …" bullet from `## Notes for the plan writer`.
  Added a D6 bullet deciding the new expectations for the two out-of-set tests that assert the old order: `livestate_verbs_test.go`'s `removeCase` dirty-refusal cells (portal, launchers and warp worktree survive, no portal or launcher permitted roots) and `mutation_record_integration_test.go`'s `Remove` dirty-refusal test (empty record, `RefusalOf` still false, header updated).
  Adjusted D6's rationale to match.
  Replaced both open risks with the one real remaining uncertainty; the `battencli` risk was dropped because no battencli integration test drives a dirty refusal.
  In the support log, resolved Q8 with those facts.
- **F2 (LOW), Goal overstatement.**
  The Goal now scopes "leaves everything as it found them" to refusals from checks that precede the archive, and states that a gate refusal or failure after the archive still leaves a tag the re-run reuses.
  Scope's first bullet now says "every pre-flight refusal" for the same reason.
- **F3 (LOW), byte-identical constraint.**
  Restated it as: the base refusal texts stay unchanged and only the reconcile suffix goes, with the accurate description of the livestate `RefusedBefore` substring dependency.
  The no-reconcile acceptance check now covers the weft refusal as well as the warp one.
- **F4 (NIT), D2 archaeology.**
  Dropped the `9de0c46d6` reference and the history sentence; D2 keeps the stale-rationale fact and why it is stale, and now names the `Remove` doc comment alongside the header.
  Logged as Q9 in the support log; Interview turn 3 already holds the history.
- **F5 (NIT), generic semantic-line-break constraint.**
  Removed the bullet.

`lyx loom validate-discussion` exits 0 after the edits.

## Deferred

None.
