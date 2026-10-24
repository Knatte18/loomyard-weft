# Board entry `record-batch-external-test-pkg`: webster record-batch fails create-not-done for Create targets declared in an external _test package

Imported from [issue #851](https://github.com/Knatte18/loomyard/issues/851).

`lyx webster record-batch` marked a batch failed with blocking `create-not-done` findings even though the Create targets existed.

What happened:
- Each finding reads `Create target "plan:<unit>#<Member>" still does not resolve: ... quarry answered not_found (the unit exists and the member is missing)`.
- The same message then says the member "is declared as `<unit>_test#<Member>`" in a `_test.go` file of that unit.
- So the member exists, in the external test package (`package x_test`), and the lookup only searched the unit `<unit>` and its files.
- It happened twice in one task, in two different batches, across three different units (batches of 6 and 5 cards). The refusals archived reports and pointed to `lyx webster recover-batch <n>`.

Why it is abnormal:
- The refusal itself shows the tool knows where the member is declared, yet still counts the target as not done and fails the batch.
- A plan that names a test in an external test package as a Create target cannot pass record-batch, whatever the implementer does.
- The recovery verb is a workaround, not a fix; each batch costs a failed record plus a recovery.

What to do:
- Make the Create-target resolution through quarry also look up the `<unit>_test` package (or resolve `Test*` members across both packages of the unit) before reporting `not_found`.
- Alternatively, make the plan validator reject or rewrite such a target to `<unit>_test#<Member>` at plan time, so the mismatch surfaces before the run.

Provenance: task `plan-card-rules`; notes `webster-refusal-record-batch.md`, `webster-refusal-record-batch-2.md`.
