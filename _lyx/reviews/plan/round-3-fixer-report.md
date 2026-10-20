# Round 3 fixer-report — plan

Every finding's premise held against the code at HEAD; all six are fixed.
`lyx loom validate-plan` returns ok after all changes, self-check included.

## Changes

- **F1 (BLOCKING), fixed as proposed, with one wording change.**
  Card 19: `startCmd`'s envelope gains top-level `mutations` (the report's `Mutations.Entries()`) and `partial`, with long help and ImpactSummary counting four new keys; `TestStart_ReconcilesTheHubFirst` asserts the empty array and a fake report's teardown mutation in `mutations`.
  Card 20: `Command`'s envelope gains the same pair in both modes; the force-case step asserts the snapshot tag in the first `--apply`'s `mutations` and the removed pair in the second's.
  Card 23: `WakeReconcile` gains `Mutations []string`, filled by the report mapping; `reconcileAtWake` logs each line at Info through `internal/logger`, keeping them out of the one-line notice; a `TestWatch` row asserts the logged lines.
  Cards 21 and 22: `reconcileRun` and `reconcileVerdict` log the report's mutations at Info.
  Card 17: the sentence is rewritten, not left as written as the fix key said, because wake has no `mutations` array; it now names orch start's and cleanup's envelopes and the wake's log lines.
- **F2 (MEDIUM), fixed as proposed.**
  Card 1: `FailedPush` carries the pair's code worktree for a `records` record, derived by trimming `weftname.Suffix` from the weft sibling's name, which covers the prime weft (`loomyard-weft` → `loomyard`); `transport`/`deferred`/`auth` name `lyx fabric push` there; `recordsConflictWayForward`'s `pushVerb` carries the push step with its directory, the git steps staying in the weft worktree, so no signature changes; `TestFailedPushRecord` pins both.
  Cards 6, 15 and 16: the records lines and exact-line rows name `lyx fabric push` in the pair's code worktree.
- **F3 (MEDIUM), fixed as proposed.**
  Card 23: Create targets `hibernatecli#wakeReconcileFromReport` and `TestWakeReconcileFromReport` added, `fabricengine#OriginReport` added to Uses; the new test's rows cover `WayForward`, `PushLockHeld`, the D3 line and the mutation lines; `TestWatch` rows assert the queued notice carries the exact way-forward line followed by `lyx resume <slug>`, `lyx resume`, or `lyx cleanup --apply` for the busy ended entry.
  Card 20: both refusal steps assert the refusal text carries the stopped branch's exact `WayForward`.
  `lyx loom validate-plan` raises no finding.
- **F4 (MEDIUM), fixed as proposed.**
  Card 9: `TestPushFailureClasses` gains prime-worktree `Fabric.PushBranch` rows for `not_fast_forward` and `auth` with `Prime` set; `TestFinalize_MergeMarkPushOrderAndVerdict` gains `not_fast_forward`-with-`Prime` and `auth` rows; `TestPublish_StuckBeforePullRequest` is added to the Edit list with `gone`, `rewound` and `auth` rows (it takes an injected `pushErr`, `internal/landingshed/publish_test.go:259`).
  Card 1: the `auth` `Prime` row is added.
- **F5 (LOW), fixed as proposed, extended by the self-check.**
  Card 9: the `Unwrap` printing rule is dropped; `commitRecordsVerb` prints `err.Error()` unchanged and appends its generic way forward only for a failure that is not a `*PushFailedError`; the Edit target `loomcli#pushFailureWayForward` becomes `loomcli#commitRecordsVerb`, since the branch now lives there; `writeHubRules` follows the same rule; `TestCommitRecordsVerb_ErrorEnvelopes` gains the two-sided `errors.Join` row.
- **F6 (LOW), fixed as proposed.**
  Card 15: the dirty line gains a Bolt form, `git commit` or `git checkout -- <path>` in the hub's `_board` worktree, then `lyx cleanup --apply`; `internal/shellguard` has no deny row for either command.
  Cards 16 and 20: the Bolt dirty rows assert that form.

## Self-check

- fix-consistency, MEDIUM: card 15's diverged line gave the prime weft `lyx fabric push` in the prime, while card 15's D3 line and the rows in cards 16 and 20 gave it `lyx board sync`. Fixed as proposed: `lyx board sync` pushes Bolt only (`internal/boardcli/cli.go:250`), so the D3 line and both rows now name `lyx fabric push` in the prime's worktree for the prime weft.
- fix-consistency, LOW, merged with fix-claims MEDIUM: cards 17 and 21 claimed run-scope logs could show a removed pair, but the run scope tears nothing down. Fixed as proposed: card 17 sends teardowns to cards 19, 20 and 23 only, and card 21 claims only moved branches.
- fix-claims, LOW: the `TestWatch` log row cannot avoid `t.Parallel` because the table loop calls it on every row (`watch_test.go:482`). Fixed with my own change: the row becomes a serial subtest that runs before the parallel loop.
- fix-affected-set, LOW: `writeHubRules`' second `bolt.Push` (after the commit, `internal/orchcli/rules.go:137`) was not covered. Fixed as proposed: card 9 now covers both refusals, each keeping its own generic way forward.
- fix-proposal, MEDIUM: `partial` was pinned to false even on an error envelope after a reconcile that changed state. Fixed as proposed: cards 19 and 20 derive it as error plus non-empty record, and card 20 adds the assertion.
- fix-proposal, MEDIUM: cleanup's `mutations` leaves out the segments' removals. Fixed with my own change: card 20 states that `mutations` carries only the reconcile's record and the segments keep their existing keys. Merging the segments into the record would rewrite segments that this task does not touch, and the review asked only that the reconcile's record reach the envelope.
- fix-proposal, MEDIUM: Info is below the default Warn level, so `logger.SetOutput` alone would capture nothing (`internal/logger/logger.go:405`). Fixed as proposed: the row also sets `logger.SetVerbosity(1)` and restores both.
- fix-proposal, LOW: in a mixed `errors.Join`, the unclassified side would get no way forward. Fixed as proposed: card 9 appends the generic line whenever any joined member is not a `*PushFailedError`, and a mixed-join row is added.

## Disputed

None.

## Deferred

None.
