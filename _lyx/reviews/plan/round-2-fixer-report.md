# Plan-Review round 2 — fixer report

Every finding's premise held against the code; all six are fixed.
`lyx loom validate-plan` over the edited plan returns `ok: true` with only the two informational `create-new-unit` notes card 9 already carried.

## Fixed

- **F1** (card 2): the Intent now says `expiredShellWarning` keeps the shell's label, signal and "outstanding for <d>", adding only the never-read clause.
  `internal/webstercli/verbs_test.go` joins `**Edit:**`, with `TestRunCmd_DiedMasterNotesExpiredShellOutcome` gaining the "never read" assertion; the grep list adds "outstanding for", and the ImpactSummary drops its test count.
- **F2** (card 6): the Intent rewords `RecoverSpawnOrAttach`'s refusal prefix to "a fork or a recovery session wrote last", and the `recovery wrote last` case also asserts the refusal does not contain "a fork wrote".
- **F3** (card 12, overview): card 12 creates `partitionedAnswers`, which classifies each ambiguous answer of a pass once.
  `LaterDeleteReferences` and `planGatePass` build its map once after `resultByTarget` and thread it; `answerSymbols`, `buildEditRegions`, `coverageSubjects`, `coveredByTarget` and `callerCoverageFindings` re-sign to take the map instead of re-reading files per call.
  The `classifier-takes-the-worktree-root` shared decision now splits the policies that visit each answer once (they keep calling the classifier with `worktreeRoot`) from the passes that consult an answer many times (they thread the per-pass map), so D6's one read per candidate file holds per pass.
  No test calls any of the re-signed functions directly, so the card's test files are unchanged.
- **F4** (card 4): `internal/websterengine/runlevel_git_test.go` joins `**Edit:**`, and `TestRun_VerifyGate` asserts through `verifyGateOf` that the entry `Run` hands `StartMaster` carries a one-line `Reprompt` opening with `Gate findings recorded at <path>` and naming the verify-fix prompt path; the unnamed helper-level row is dropped.
- **F5** (card 9): the classifier table gains a row: a `//go:build linux &&` candidate beside a `//go:build !linux` file is not partitioned.
- **F6** (card 6): `internal/websterengine#ArchiveRunAfterReset` joins `**Edit:**`, and the Intent rewords every "a fork wrote last" doc comment on the card's symbols, that one included, to "a fork or a recovery session wrote last".

## Disputed

None.

## Deferred

None.
