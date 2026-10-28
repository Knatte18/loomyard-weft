```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (per change: set found, covered or not)**

1. **`MatchPath` gate blanking (`refscanner.go`)**
   - Callers: `refscanner.go:120` (the audit scanner), `planparser/fabricref.go:59` and `:74` (the plan check, which shares the rule), and `refscanner_test.go:107`. Covered: the four new rows, the `fabricengine/doc.go` paragraph and the `ReferenceRule` doc.
   - Not covered: no `planparser` test or doc line for the changed behaviour (finding F-A1).
2. **`batchFabricReferenceEvidence` split out of `AcceptBatchFabricReference`**
   - Callers: `webstercli/acceptaudit.go:133`, `recoverbatch.go:511`, and `acceptbatch_test.go` (lines 65 to 193).
   - The refusal texts are unchanged except that the `rerun` argument is threaded through; the old string `re-run "lym webster <verb> N"` is rebuilt identically. Covered.
   - Nil `readOnly` now refuses instead of panicking. No caller passes nil to `AcceptBatchFabricReference`; `recoverbatch.go:511` may pass `deps.ReadOnly` nil, so this is safer. Covered.
3. **`acceptAuditWayForward` and the needs-fresh text**
   - The old route text is gone from all non-test code (grep for `every recorded command is read-only` and `batch's start commit (` finds nothing outside the diff).
   - `webster/doc.go` is updated. The Master stencil is out of scope per the record.
   - No other stencil or yaml mentions `accept-audit` or `needs_fresh` text (grep of non-`.go`, non-`_lym` files finds only `docs/research` background). Covered.
4. **`PlanReset` run-lock text**
   - `runlevel.go:408` keeps its own "wait for it to finish" text for `run`, a different verb and not in scope.
   - No other test asserts the old `PlanReset` text (grep finds only an unrelated `shedengine` string). `doc.go` is updated. Covered.
5. **PATTERN and commit-message checks**
   - `PATTERN-refusal-way-forward` is met: the changed refusals name steps, and tests reach them.
   - There are no declared PATTERN changes and no `Pattern-Departure` trailers. Nothing to flag.

**Findings**

- **F-A1 | LOW | consistency | `internal/planparser/fabricref.go:74` (and `:59`)**
  - The summary claims the plan check no longer flags a Verify command naming its own `-weft`-suffixed card ID. The card-ID check at `:74` still rejects the ID itself, with the text "every command naming the card fails the audit".
  - After this change that text is false for `lym gate` commands, so the two parts disagree. No `planparser` test pins the shared-rule effect.
  - The claim and the message are both slightly off; the minimum fix is to narrow the message wording, or drop the claim from the summary.
  - Evidence: `fabricref.go:59-75`, `summary.md` "The plan check uses the same rule".

**Tool calls (3)**
1. Grep for the old texts, `AcceptBatchFabricReference` callers, and `MatchPath` and the patterns: to find consumers beyond the diff.
2. Grep for `ErrRunBusy`, `run.lock held` and `accept-audit` across non-`.go` files: to find stencils and docs that assert the changed text.
3. Grep for "wait for it to finish" in tests, and a read of `planparser/fabricref.go`: to check the old-text assertions and the plan-check consumer.
