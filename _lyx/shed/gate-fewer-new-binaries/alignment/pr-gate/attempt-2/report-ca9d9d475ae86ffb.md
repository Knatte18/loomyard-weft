---
verdict: APPROVED
---

## Measure

The board entry `gate-fewer-new-binaries` asks for four things: verify runs the unit tier once, a re-run on an unchanged tree reuses passed results, a verify after a failure runs the failed test first, and the before and after numbers on OSL-1046 go into `docs/benchmarks/gate-test-xdr.md`.
The decision record's `Added after Discussion (parent, 2026-10-20)` entry amends it: the loom plan's `## verify:` also runs the integration tier alone, through `pattern/PATTERN-test-speed.md`'s Webster gate, Publish and Finalize row and the `internal/languageconfig` doc's example `verify_tiers`.
The diff of pull request 768 at `2964ff71` is measured against that amended entry.

Points the diff meets:

- Unit tier once: `CLAUDE.md`'s push rule names the single integration run, `PATTERN-test-speed.md:28` and `internal/languageconfig/doc.go` give the plan verify as build, vet under each tag, then the integration tier alone, and `internal/impactset` drops the untagged step from the impacted-set command that the Webster-Burler-Fix round gate runs.
  The board's note names the loom verify gates beside `darn.yaml`, so the impacted-set change is inside scope.
  This settles attempt 1's rejection.
- Cache-stable re-runs: `lyx gate test` strips `LYX_TRACE_ID` from go test's environment (`internal/gatecli/run.go`), a tagged hub run exports a content-addressed prebuilt path from the pool's store (`internal/gatecli/prebuild.go`, `gateslot.Pool.PrebuiltDir`), and the `internal/shedbuild` test no longer writes into its package directory.
- Failing tests first: `internal/loomshed/verifygate.go` puts the record's failing tests before the told or derived command and the `tmux` package pass after it, in the code path both forms share.
- Measurement: `docs/benchmarks/gate-test-xdr.md` gains the re-run section with before and after rows on OSL-1046, with the packages run and cached per run.

The `logger.TraceIDEnv` constant and its use in `internal/reedengine/lifecycle.go` and `internal/shedverbs/loop.go` follow from the trace-id change and are inside scope.
`mill-config.yaml`'s `done_gate` still names both tiers, but no Go code reads that file, so it is no verify gate of this entry.

The record's open risks, other per-invocation inputs and tests that open rewritten files, name no parent correction and do not escalate.

## Overlap

`lyx loom peers` lists no peer runs and no parent commits (`own_row` 21), so there is no overlap and no landing order to give.

## Board entries touching this run

- `trace-id-env-literal` stays open: the three test files it names still spell `LYX_TRACE_ID` as a literal, so this run does not make it obsolete.
- `verify-diff-scope` and `bench-osl1046-xdr` cover tree-scoped verify and a `go_parallel` sweep, work this diff does not do.
- `darn-verify-parent-fix` and `darn-verify-tier-unrunnable` change other parts of the verify loop than this diff does.
