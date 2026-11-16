```yaml
lens: code-affected-set
outcome: reported
```

Per change — the affected set found — covered or not:

- **Moved files/symbols** (report, fingerprint, suspect, digest, cardgate, awaitbatch, conflictrecord, verifyparse, verifygatereport, state, partition, planbaseline, strand): docs naming old paths are in `docs/research/...` only (measurement reports, exempt); `docs/overview.md` gains a runplan row and move paragraph; `websterengine/doc.go` references updated — covered.
- **Exported webster names kept as aliases** (`internal/websterengine/runplan.go`): all importers compile; test helpers in loomcli, loomshed, webstercli, failuretrace, tokencount nest `State: runplan.State{...}` — covered.
- **`cmd/lym/testonlycode_linux_test.go` allowlist**: drops `websterengine.Fingerprint`/`WriteReport`, now variables pointing at runplan functions production uses — covered.
- **Sentinel prefix change `webster:` → `runplan:`** (`ErrFingerprintMismatch`, `ErrPlanConflict`, `ErrCardNotDone`, `ErrPartitionMismatch`, `ErrLaterTargetRemoved`, FindBatch refusal): no test, stencil or contract asserts the old text (handler grep plus mine over `contracts/`, `docs/`, `pattern/`, `cmd/`) — covered.
- **Way-forward text now caller-passed** (PlanConflict via `planConflictWayForward`, later-target via `laterTargetHandBackWayForward`, partition via `partitionMismatchWayForward`, FindBatch/AwaitBatch via `findBatchWayForward`, suspect plan path via `planPathClause`, verify findings via `verifyGateWayForward`): each produces the same string the old code built inline — covered.
- **Guard-test package lists in `cmd/lym`**: `checkedcall_test` (unlisted package = pinned at zero raw sites; runplan has none), `rawgitmutation_test` (scans websterengine only; runplan runs git only through the `Git` interface), `configstrictness_test` (runplan doesn't call `LoadOrTemplate`) — covered.
- **PATTERN-cwd-resolution "Bound packages" list**: lists `websterengine`; code moved out of it into runplan, which is not added — not covered (F1).
- **runplan package doc** (`internal/runplan/doc.go`): claims nothing names an implementer's verbs; moved code contradicts — not covered (F2).
- **Decision-record constraint PATTERN-lym-files for the shared plan-conflict path**: not named by lymdirs — not covered (F3).
- **Declared PATTERN changes**: directive declares none; no `Pattern-Departure:` trailers in the range's commits — nothing to check.

Findings:

- **F1 — LOW, consistency** — `pattern/PATTERN-cwd-resolution.md:38` — The "Bound packages" list names `websterengine`, so the engine code this diff moves into the new `internal/runplan` leaves the list without runplan being added. Evidence: `pattern/PATTERN-cwd-resolution.md:36-38`; runplan imports `lymdirs` but not `lymcwd` (`internal/runplan/suspect.go` imports), so the code complies, only the rule document lags.
- **F2 — LOW, consistency** — `internal/runplan/doc.go:16` — The package doc says "Nothing here names an implementer's verbs", yet the shared conflict record header spells webster's verb and the shared bind trace logs under webster's name, so under Weaver both name webster. Evidence: `internal/runplan/conflict.go:142,172` (`"# Plan conflict at begin-batch %02d"`, written and parsed); `internal/runplan/postbatch.go:163,166` (`logger.Info("webster: handle bind", ...)`). The change description keeps both deliberately for the pinned log line and on-disk format; the doc claim must state that exception or the text must come from the caller.
- **F3 — MEDIUM, design** — `internal/runplan/conflict.go:122-125`, `internal/runplan/state.go:11` (`stateMutateLockName`), `internal/runplan/fingerprint.go:35` (`PlanBaselineDirName`) — The plan-conflict file name is now a shared constant declared in runplan (with the routed-conflict name, the mutate lease and plan-baseline dir), while the decision record's Constraints line states "`_lym/weaver/`, `.lym/weaver/` and the shared plan-conflict path are named only by `lymdirs`" and PATTERN-lym-files says a file is "named only by `lymdirs`". These were already webster-private names before the diff, but the diff is the step that makes them shared, and lymdirs carries none of them (grep of `internal/lymdirs/` for these names found nothing). Evidence: `_lym/discussion/decision-record.md` Constraints (PATTERN-lym-files bullet); `ConflictRecordName = "plan-conflict.md"` in `internal/runplan/conflict.go`.

Tool calls:
1. Grep `PATTERN.md`, `pattern/`, `docs/`, `contracts/`, `cmd/` for moved file paths, symbols and old sentinel text — context had no search outside the diff.
2. Grep `pattern/` for websterengine/runplan mentions, `pattern/` + `internal/lymdirs` for moved file names, and tests referencing websterengine — needed the rule documents and guard tests.
3. Read cwd-resolution "Bound packages" list and checked `cmd/lym` tests for an enforcing list — needed what the list governs.
4. Read `checkedcall_test`, `rawgitmutation_test`, `configstrictness_test` package lists — needed whether runplan must join them.
5. Grep cwd-resolution background and `cmd/lym` tests for "bound" enforcement — needed whether a test pins the list.
6. Read the head of PATTERN-cwd-resolution and `pathrules_test` use of websterengine — needed the rule's scope.
