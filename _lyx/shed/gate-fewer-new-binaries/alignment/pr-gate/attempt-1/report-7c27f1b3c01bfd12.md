---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: pattern/PATTERN-test-speed.md:28
    summary: The loom verify gates still run the unit tier twice, a point of the board entry the diff leaves out
    fix: In pattern/PATTERN-test-speed.md line 28, change the row for Webster's gate, Publish, Finalize so the plan's verify runs build, vet under each tag, then the integration tier alone, stating that the integration tag runs every untagged test too (the same reason CLAUDE.md line 37 now gives), and drop the untagged tier from it. Then reword the decision record's Scope bullet so it names the loom plan verify beside darn.yaml and CLAUDE.md. Before committing, grep contracts/stencils/loom and contracts/specs for any other text telling a plan writer to put a plain go test run beside the integration run in the plan-level verify section, and align it the same way.
---

## Measure

The board entry `gate-fewer-new-binaries` asks for four things: verify runs the unit tier once, a re-run on an unchanged tree reuses passed results, a verify after a failure runs the failed test first, and the before and after numbers on OSL-1046 go into `docs/benchmarks/gate-test-xdr.md`.
The decision record has no `Added after Discussion` entries, so the board body is the measure.
The darn.yaml verify on Bolt already reads `lyx gate test --tags integration ./...`, as the record's auto-mode assumption expects of the parent.

Points the diff meets:

- Cache-stable re-runs: `LYX_TRACE_ID` is stripped from go test's environment (`internal/gatecli/run.go`), the tagged run's prebuilt `lyx` gets a content-addressed path in the hub store (`internal/gatecli/prebuild.go`, `internal/gateslot/pool.go`), and a shedbuild test stops writing into its package directory.
- Failing tests first: `internal/loomshed/verifygate.go` puts the record's failing tests before the told or derived command and the `tmux` package pass after it, in the one code path both forms share.
- Measurement: `docs/benchmarks/gate-test-xdr.md` gains the re-run section.
- Unit tier once, for darn and the push rule: `CLAUDE.md` names the single integration run.

The `logger.TraceIDEnv` changes in `internal/reedengine/lifecycle.go` and `internal/shedverbs/loop.go` come from Darn-Review's F4 finding and its self-check, which trace back to the trace-id change, so they are inside scope.
The carried-over open risk `store-race-error-paths-untested` was fixed in `a13736fe` and `919e62db`, and it names no parent correction, so it does not escalate.

### [BLOCKING] The loom verify gates still run the unit tier twice

**Location:** `pattern/PATTERN-test-speed.md:28`

**Issue:** The board entry's note `verify-unit-tier-twice` says: "The verify command of `darn.yaml` (and the loom verify gates) is `lyx gate test ./... && lyx gate test --tags integration ./...`", and its idea is that verify runs only the `integration` tier.
The decision record's Scope narrowed this to darn.yaml and `CLAUDE.md`, saying `CLAUDE.md`'s push rule is "the repository text that names the two tiers".
`PATTERN-test-speed.md:28` is a second such text, and the run edited the row next to it.
Its row for Webster's gate, Publish and Finalize still says the plan's `## verify:` runs "build, vet under each tag, the untagged tier, then the `integration` tier".
Plan writers follow that row, as the last three loom plans on Bolt show: `hub-workspaces` (`f5e33a1a1`) and both `orch-intake-fork` plans (`bcc9f3db6`, `ea73d17ba`) each run a plain `go test` over their packages and then `go test -tags integration` over the same packages.
So every loom run's Webster gate, Publish and Finalize still link and run the unit tier twice, which is the double run the entry asks to remove.
The decision record's own rationale, that no test is excluded by the `integration` tag, applies to the plan verify unchanged.

## Overlap

`lyx loom peers` lists no peer runs and no parent commits (`own_row` 21), so there is no overlap and no landing order to give.

## Board entries touching this run

- `trace-id-env-literal` was filed from this run's Darn-Review for three test files the diff leaves untouched (`internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go`, `internal/testkit/tmuxkit/tmuxkit_integration_test.go`), so this run does not make it obsolete.
- `verify-diff-scope` and `bench-osl1046-xdr` are ordered after this task; the diff does neither one's work.
- `darn-verify-parent-fix` and `darn-verify-tier-unrunnable` change other parts of the verify loop than this diff does.
