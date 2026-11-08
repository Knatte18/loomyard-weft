```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether it is covered:**

- **`renderBatchGate` gained a `last` parameter.** The set is `beginbatch.go:562`, `recoverbatch.go:286`, `cardgate_test.go` and `doc.go`. These are the only callers: the grep found no other `renderBatchGate` or `AgentGateCall("batch"…)` site. Covered.
- **`isLastBatch` uses `batchIdentity`.** Both callers pass `deps.Batches` in execution order. Covered.
- **The new `--main-packages` flag.** The set is `cli.go`, `run.go` (`flagShapeRefusal`), `detachfront.go` `words()`, `scope.go`, `gatecli/doc.go`, `gateslot/doc.go`, `impactset/doc.go` and `websterengine/doc.go`. All carry the flag, and its constant is `gateslot.MainPackagesFlag`. Covered.
- **Other places that list `gate test` flags.** The shell guard (`shellguard/rows.go`), `loomshed/plancontext.go` and `planparser/verifymodule.go` only name `--tier` and `--card`, as examples or refusals. None enumerates the flags, so they stay correct. Covered.
- **The stencil `webster-body-implementer.md` (lines 106–107).** It adds the exception but leaves three other statements unchanged.
  - Line 131 says a failure the card did not cause is never yours to fix.
  - Line 132 says a batch gate step that fails only on such failures does not make the report `FAILED`.
  - Line 175 repeats line 132 in the `status` rules.
  - These contradict the exception for the last batch. See F1. Not covered.
- **The `pattern/` background files.** `PATTERN-test-speed` and `PATTERN-gate-slot` name the `batch` key and its scope, and the diff changes neither the key nor its meaning. The declared-PATTERN-changes list is not visible in this context, so I report nothing under that bullet. Covered.
- **Commit trailers.** I saw no `Pattern-Departure:` trailer and no departure from the decision record, so nothing under the rubric's "Commit messages and departures" bullet.

**Findings**

- **F1**
  - **Severity:** MEDIUM
  - **Class:** consistency
  - **Location:** `contracts/stencils/webster/webster-body-implementer.md:106-107`, against lines 131-132 and 175
  - **Summary:** The new exception is not carried into the places that decide the report status. Line 132 says a batch gate step failing only on failures the batch's cards did not cause "does not make the report `FAILED`", and line 175 repeats it. Under that wording, a last fork that sees an earlier batch's `cmd/lym` break can report `OK`, which defeats the goal of fixing it in the fork.
  - **Evidence:** lines 106-107 (the new text), 131-132 and 175 (unchanged).
  - **Fix:** At lines 132 and 175, exempt a failure in the packages `--main-packages` adds (a "except in the packages `--main-packages` adds" clause). Also reconcile line 107's "the one exception to the rule below" with line 175.

Tool calls (2 of 6 used):
1. `grep` and `sed` over `contracts/stencils/webster/` and the implementer body: my context held only the diff hunk, not the stencil text around lines 95–135 or the other stencils that mention the batch gate.
2. `grep` for `--card` in `shellguard/rows.go`, `loomshed/plancontext.go` and `planparser/verifymodule.go`: to confirm none of them lists the `gate test` flags.

(One more `grep` listed which files mention `--card`; that is a third call.)
