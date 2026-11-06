```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I list what reads or asserts it and whether the diff covers it.

**Per change, the set found**

1. **New cause `plan_order` (`ConflictCausePlanOrder`, `PlanConflictError.Error`, `BeginBatch`)**
   - Readers of `Cause`: `WriteConflictRecord` (`conflictrecord.go:42`) and the `webstercli` envelope.
     - Both copy the string through, so a new value is safe. Covered.
   - Consumers of the record: `ReadConflictRecord`, `ClaimConflictRoute` and `planfixer.go`.
     - None switches on the cause. They read the batch and findings only. Covered.
   - Docs: `webstercli/beginbatch.go` and `websterengine/doc.go` are updated. Covered.
   - Tests: `webstercli/verbs_test.go:411` and `shedadapters/webster_test.go:205` only use `plan_defect`. Nothing breaks. Covered.
   - Master stencil's cause list (`webster-template-master.md:238`) is updated. Covered.
   - The plan-fix stencil and the master stencil's quote of the way forward are not updated. See F1 and F2.
2. **Way-forward text of `PlanConflictError.Error`**
   - The master stencil line 265 quotes the old text verbatim. Not covered. See F2.
3. **`planindex.CheckDeleteBeforeReference`**
   - All three emit sites in `deleteorder.go` use it, and `beginbatch.go:503` compares against it. Covered.
   - Literals remain only in test files and comments (`deleteorder_test.go:240`, `planfixture_helper_test.go:106`). These are not production sites.
   - No PATTERN entry covers it, and the commit declares no PATTERN change or `Pattern-Departure` trailer. Nothing to check.
4. **Spec and doc prose** (`loom-plan-checks-spec.md`, `planglyph/doc.go`, `websterengine/doc.go`)
   - The three agree with each other. No other spec names the causes (`grep` of `contracts`). Covered.

**Findings**

- **F1 — MEDIUM, design.** `contracts/stencils/webster/webster-template-master.md:238-242` sends `plan_order` through the fix-plan ladder, but the plan fixer's stencil was not changed.
  - Evidence: `webster-template-plan-fix.md:9-11` says cards "name targets the tree no longer holds" and that the job is to "match the tree, when that leaves what each card does unchanged".
  - Evidence: its "Decide" section says a finding that "would add, drop or reshape the card's work" is `plan_writer`.
  - Moving a delete to a later card is that kind of change, so the fixer returns `verdict: plan_writer` every time. Each `plan_order` refusal then costs a fixer spawn for nothing before the run ends stuck.
  - The master stencil also ends its `plan_order` sentence with a colon, which attaches the move-the-delete instruction to the fixer steps.
  - Fix: either give the fixer stencil a `plan_order` rule (it moves the delete and rebaselines, or states that it hands the finding back), or have the master stencil send `plan_order` straight to stuck, with no fixer.
- **F2 — LOW, consistency.** `webster-template-master.md:265` still quotes the refusal as "edit the plan so the named cards match the tree, run … rebaseline … then begin-batch NN again".
  - A `plan_order` refusal now reads "move each named delete to a card after the card that still references it, run …" (`beginbatch.go:114-119`).
  - The stencil's quote is correct only for the other two causes.
  - Fix: rephrase the line to describe the refusal without quoting one wording, or note that a `plan_order` refusal names moving the delete.

**Tool calls (2 of 6)**
1. A `grep` of `contracts` and `_lym/specs` for the cause names and "match the tree", plus a read of the master stencil lines 225-262. My context lacked the other stencils and specs that name the causes.
2. A read of `webster-template-plan-fix.md` lines 1-60 and master lines 262-270. My context lacked the fixer's decision rules and the quote at line 265.
