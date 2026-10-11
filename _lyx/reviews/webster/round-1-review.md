---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: tools/tokencount/count.go:953
    summary: The left-out line reads "1 session transcripts were left out" for a count of one, and time_integration_test.go pins that wording
---

Range: `status.json` gives `product.parent` `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the seven card commits.
`lyx gate test` passes for `./tools/tokencount`, `./internal/loomcli`, `./internal/battenshed`, `./contracts/stencils` and `./cmd/lyx`.
`lyx gate test --tags integration` passes for `./tools/tokencount`, and for `./internal/loomcli -run TestLoomFailedReedUpRefuses`.
I did not run the tmux-tier `TestSmokeStepBootstrapWiring` subtest; I read it against card 5.

Each focus question, settled against the diff:

- **Cards 1 and 2.**
  `startsTurn`, `stallCap` and `workingIntervals` match card 1, including the `IsCompactSummary` exclusion, whole-interval idling and the report sentence rendered from `stallCap`.
  `CountRun` skips a main-session transcript before `latestTitle`, `countFile`, `recordBase` and `readMerriamStart` read it, so the bound has no effect on forks, Merriam starts, `BaseSHA` or `First`/`Last`.
  `runBound` takes the newest tag for an in-flight run, `tags[1]` for a finished one, and the zero time otherwise.
  The `main.go` package doc was updated for both cards.
  `run` reads the worktree `os.Stat` once and passes `inFlight` to `runBound` and to the shed-record branch.
  No new git call site.
- **Cards 3, 4 and 5.**
  Each changed refusal has its spec row: the two-route `seed missing` row, the other-run-id `seed missing` row, `run: status file missing`, and `start/step: seed disagrees`, plus the lead sentence.
  Each has a reaching test.
  `TestResolveRunID` covers the `self` and other-run-id branches, `TestVerbRefusals`'s `Run_SeedMissing` row reaches `loomPreRun`'s status-file refusal, and `TestLoomFailedReedUpRefuses` and `TestSmokeStepBootstrapWiring` reach the disagreeing-seed refusal.
  Recovery text names only `lyx loom start`, `lyx loom step`, `lyx shed seed` and a plain `rm <path>`; no `_lyx` path is spelled.
  `disagreeingSeedRefusal` wraps `shedrun.ErrDisagreeingSeed` with `%w` and never holds "keep the existing seed".
- **Card 7.**
  `conditionText` leads the quiet, api-error and driver-dead notices with `haltText(status); `, and a state-changed notice stays `haltText(status)` alone.
  The `noticeDriverBusy` path is untouched, so it does not gain a second state prefix.
  `notice.go`'s two quiet texts drop `while the child is running`, and `battenshed/doc.go` carries the matching sentence.
- **Test economy.**
  The diff adds one top-level test, `TestLoomVerbHelp_NamesBothSeedingRoutes`, and `internal/loomcli`'s row in `test-budget.yaml` rises by one with it.
  The other coverage extends `TestRunTimeByRole`, `TestCountRun`, `TestRunReadsArchivedShedStatus`, `TestResolveRunID`, `TestVerbRefusals`, `TestLoomFailedReedUpRefuses`, `TestSmokeStepBootstrapWiring`, the three `TestNotice_*` tests and `TestStencilClaims`'s table.
  The `tokencount` row is unchanged.
  Every added behavior has coverage.
- **Per-card mechanical checks.**
  The cards carry no `Rename` or `Delete` group, so no AST-script or `assert-no-callers` check applies.
  The `CountRun` re-sign compiles, and its callers all pass the zero time or the computed bound.

### [NIT] Left-out line is ungrammatical for a count of one

**Location:** `tools/tokencount/count.go:953`
**Issue:** `"%d session transcripts were left out: ..."` renders "1 session transcripts were left out" when `LeftOut` is 1.
`TestRunReadsArchivedShedStatus` pins that rendering twice.
**Fix:** Optional: word the line so it reads for any count, for example `Session transcripts left out: %d; they started before ...`, and update the two assertions in `time_integration_test.go`.
