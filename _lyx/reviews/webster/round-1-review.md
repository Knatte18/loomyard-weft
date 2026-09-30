---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loggerconfig/config.go:60
    summary: time.Duration(days) * 24 * time.Hour overflows for trace_retention_days above 106751, yielding a negative MaxAge whose cutoff lies centuries in the future, so the exit sweep deletes every non-live trace group
  - id: F2
    severity: LOW
    location: internal/logger/retention.go:1
    summary: New and changed Go comments across the diff break lines mid-phrase at a fixed column instead of at sentence or independent-clause boundaries, against CONSTRAINTS.md's Comment Line-Break Convention
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`, with `main` taken from `_lyx/shed/trace-log-retention/status.json`'s `product.parent`; merge base `85902381e`, five commits, 20 files.

Evidence gathered: `go build ./...` passes, `go test ./...` passes in full, and `go test -count=1 -tags integration` over `internal/logger`, `internal/loggerconfig`, `internal/configreg` and `cmd/lyx` passes.
A throwaway probe test (deleted afterwards, tree left clean) loaded a `logger.yaml` carrying `trace_retention_days: 1000000` and got `err=nil`, `MaxAge: -1620477h52m48s`, and a cutoff of `2211-08-12`.

Focus items settled:

- `sweep()` matches card 1: files group by the captured trace id, activity is the newest mtime, a failed stat falls back to the filename timestamp without aborting, and one live-pid file keeps the whole group out of both the age cut and the count budget.
- The D3 order in `notifyExitAndSweep` holds on every path: `NotifyExit`, then `CurrentSinkArmState`, then an early return when not armed, with no config load before the check; the redirected branch returns the defaults without calling `Load`.
  The Warn in `exitSweepBounds` is emitted while the sink is armed, so it lands in the trace file and, at the default Warn level, on stderr; `TestExitSweep_InvalidConfigWarnsAndKeepsExitCode` checks the output half.
- `sinkDir` and `sinkAnchorPath` are written only in `armDurableSinkLocked` and `resetDurableSinkLocked`, both under `sinkMu`, and read only in `CurrentSinkArmState` under `sinkMu`; `Armed` is `sinkArmed && sinkOK`, and the four card-4 cases exist and pass, including the redirect-after-arm case.
- The tests card 1–5 list all exist and exercise their claims: mtimes go through `os.Chtimes`, each invalid fixture carries both keys with only the key under test changed, and the `--help` subtests cover both the absent and the invalid `logger.yaml`.
- Per-card mechanical checks: `Sweep`'s only production caller is now `cmd/lyx/exitsweep.go`, `resolveSinkDirLocked`'s callers are both in `sink.go`, the degrading set is updated in both `configstrictness_test.go` and `CONSTRAINTS.md`, and `retentionAgeBound`, `retentionCountBound`, `retentionCandidate` and `minInt` are gone with nothing equivalent to `RetentionBounds` pre-existing.
- No doc still states the retired rules: `logger.go`'s durable-sink section and `docs/shared-libs/README.md` both describe the exit sweep; no `manifest/designs` or other `docs/` page describes the sweep.

### [MEDIUM] Large trace_retention_days overflows MaxAge and deletes every non-live trace

**Location:** internal/loggerconfig/config.go:60
**Issue:** `Load` validates only that `trace_retention_days` is a positive `!!int`, then computes `time.Duration(days) * 24 * time.Hour`.
For any value above 106751 (`math.MaxInt64` nanoseconds is about 106751.99 days) the product wraps around `int64`.
With `1000000` the result is a negative `MaxAge`, so `sweep`'s `now.Add(-bounds.MaxAge)` lands in the year 2211, every non-live group is `Before(cutoff)`, and the exit sweep deletes all of them.
An operator writing a very large number to mean "keep effectively forever" gets the opposite: every dead trace goes on the next armed exit, with no error and no Warn.
Other large values wrap to an arbitrary positive or negative duration, so the cutoff is unpredictable.
**Fix:** Reject a `trace_retention_days` value above the largest day count a `time.Duration` can hold, with an error naming the key, so the exit hook Warns and falls back to the defaults; add a `config_test.go` case for an over-range value.

### [LOW] New and changed comments break mid-phrase instead of at semantic boundaries

**Location:** internal/logger/retention.go:1
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line in new and changed Go comments, with further breaks only at independent-clause boundaries.
Many comments this diff adds or rewrites are wrapped at a fixed column instead, breaking mid-phrase, for example:
`retention.go`'s header ("the newest mtime / among its files", "Sweep takes a / directory path"), `RetentionBounds`' doc ("may / go without activity"), `DefaultRetentionBounds`' doc, and `Sweep`'s doc ("only the first / bounds.Count survive");
`sink.go`'s `resolveSinkDirLocked` doc ("reports whether a directory / could be resolved") and `CurrentSinkArmState`'s doc;
`logger.go`'s rewritten sweep paragraph ("the newest / mtime among its files", "never counts / toward");
`exitsweep.go`'s header ("every zero-exit run / that logged nothing else") and `exitSweepBounds`' doc;
`loggerconfig/config.go`'s package doc and `positiveInt`'s doc, and `template.go`'s header and `ConfigTemplate` doc;
`main.go`'s rewritten exit-hook comment in `main`;
and the new test comments in `retention_test.go`, `sink_test.go`, `main_integration_test.go`, `config_test.go` and `configsync_test.go`.
No width finding is made; only break placement.
**Fix:** Rejoin each affected comment so every sentence sits on one line, breaking further only at a semicolon or a comma plus coordinating conjunction that starts an independent clause.
Untouched legacy lines, such as the rest of `isLyxWorktree`'s paragraph around its one reworded line, stay as they are.
