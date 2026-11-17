```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran each set-premise finding's search against the change set. I found no uncovered members.

Findings checked (all no gap):
- **F1:** `stop` is built at three sites in `run.go` (`stopKindBatch` at the ladder-exhausted branch, the failed-stop branch, and `nextAttemptRung`). The ladder-exhausted branch is not a failed-strand-stop path, and the fixer stop path already uses `stopKindVerify` via `verifyStop`. The resume path (`resume.go:328`) already derives its kind from `nextAttemptRung`. It never records a stop for a failed strand stop; it returns the stop error text as the cause and leaves the attempt in flight, so the next call retries the stop.
- **F2:** `settleRecordedStrands` has one caller, `Reset` (`reset.go:132`), and it was updated for the new return value.
- **F7:** `weaverengine.RunActive` is still defined and tested; `Reset` no longer calls it. The only other `RunActive` callers are in webster, a separate package.
- **F8:** `nextAttemptRung` has two callers (`run.go:516` and `resume.go:324`). The new `outcomeFault` case covers both. `outcomeFault` is produced only in `record.go:100`.
- **F9:** No remaining text says a batch begin clears the stop record. `doc.go:77`, `record.go:77` and `resume.go:493` all say only a done batch or a passing verify clears it, and no spec under `contracts/` says otherwise.
- **F10:** `weaverengine` is now in the bound-packages list. It is also already in the `PATTERN-config-layers` degrading list; no other `pattern/*.md` file lists engines.

Findings left unchecked: F3, F4, F5, F11 and F12 are coverage-gap findings, not set premises. F6 has a single site, `stopReason` via `truncateCause`.

**LOW — `run.go:585` `nextAttemptRung` fault case:** a fault retry is not counted by `lostOnRung`, which counts only lost attempts. A persistent fault therefore retries on the same rung on every re-step, with no allowance. Each call still stops with a stuck reason, so the loop is not unbounded within a call. This matches the review's "no allowance" wording; I record it only as a behaviour the reader should expect.

Tool calls (4, all Bash greps):
1. `settleRecordedStrands`, `nextAttemptRung` and `RunActive` callers: for F1, F2, F7 and F8 coverage.
2. Stop-record clearing wording across `internal`, `contracts` and `pattern` (F9): the context held only the files the fixer touched.
3. `PATTERN` list membership and `stopKind*` sites (F10, F1): the context lacked the other pattern files and the full stop-kind site list.
4. `outcomeFault` and `StopRecord{` construction sites: the context lacked the repo-wide count of these sites.
5. Spec search under `contracts/` for stale begin-clears-stop or fault-climbs text (F9, F8): the context lacked the contract files.
