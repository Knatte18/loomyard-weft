# Webster round 1 — fixer report

All five findings were validated against the code and the plan and fixed, each in its own commit.

## Fixed

- **F1** (BLOCKING) — `7bab3ad43`.
  Deleted the duplicated `logSink` type and `captureLogs` helper from `cmd/lyx/stencilseed_integration_test.go` and `internal/hubreconcile/ensure_integration_test.go`;
  their callers now use `logcapture.Capture(t)` from `internal/testkit/logcapture`, per `PATTERN-testkit`.
- **F2** (MEDIUM) — `cc396cc82`.
  `TestRunCLI_ReconcileBackfillFailureFailsTheVerb` became the table-driven `TestRunCLI_ReconcileBoardFailureFailsTheVerb`, still one top-level test, so the budget row is unchanged.
  It keeps the original row (unreachable remote: heal skipped, recorded binding's push fails) and adds three: a rejected push after a landed heal, which also fails the `present` binding's push;
  a rejected commit of the heal;
  and a rejected commit of the recorded binding.
  Each row asserts a non-zero exit, the kept `pairs` key, the error naming the first failure, and each side's detail.
  `TestRunCLI_AdoptRemoteWeftScenario`'s no-origin step asserts the non-zero exit again instead of ignoring it.
- **F3** (MEDIUM) — `7ed490491`.
  `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` gained two subtests:
  a rebase conflict with only non-seed commits ahead returns `ErrPullRebaseFailed` with the tip unchanged and nothing recorded;
  and a drop followed by a retry the remote's pre-receive hook rejects returns the retry's own error, keeps the drop, leaves a clean tree with no rebase in progress and records one `commits_dropped` entry naming the seed.
- **F4** (LOW) — `f4f837d1b`.
  Took the review's first option, which card 10's Intent asks for ("a board push that lands the commit after a transport failure clears it"):
  `reconcileHubWide` now pushes the board whenever the pull-first did not skip, whether or not this run committed, so an earlier run's unpushed hub-wide commit is pushed, and a failed push keeps reporting `skipped` on every rerun.
  The Warn now reads as a board push failure and carries `committed`;
  the function comment and `doc.go` state the rule.
  `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` gained a rerun under the still-rejecting remote that asserts no new commit and no stamp.
- **F5** (NIT) — `63dd93d8e`.
  The comment in `internal/reedcli/smoke_switchseed_test.go` now names the three-way `ModeFor` and drops the false claim about unstamped binaries.

## Verification

`lyx gate test` passed for `./cmd/lyx`, `./internal/fabriccli`, `./internal/fabricengine`, `./internal/hubreconcile` and `./internal/reedcli`, without tags and with `--tags integration`.
`go vet` passed for the edited packages under their tags, and `lyx loom lint-comments --base a9bf0d982` reported no findings.
The module-wide runs and the tmux tier are refused in this session, so the F5 comment change was only vetted under `-tags tmux` and not run.

## Disputed

None.

## Deferred

None.
