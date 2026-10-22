# webster record-batch refuses with create-not-done: quarry not_found for planned test targets

`quarry-external-test-targets` · Note · bug, webster, quarry

Imported from [issue #823](https://github.com/Knatte18/loomyard/issues/823).

What happened:
`lyx webster record-batch` refused twice in one task, for batches `01` and `09`.
Both refusals are `create-not-done[blocking]` findings of the same form: `Create target "plan:<pkg>#<TestName>" still does not resolve ... quarry answered not_found (the unit exists and the member is missing)`.
Batch `01` listed four unresolved targets across two cards (three in one package, one in another).
Batch `09` listed one target, whose package path ends in `_test` (`internal/loomcli_test#...`), the form of an external test package.
Each refusal archived a report under `_lyx/webster/reports/` and pointed at `lyx webster recover-batch <n>`.

Why it is abnormal:
The refusal cannot distinguish a card that never wrote the test from a quarry lookup that fails to find a test that exists.
The `_test`-suffixed package lookup in batch `09` is a candidate for the second case: the lookup may resolve the directory unit but not members of an external test package.
Two batches failing the same check in one task suggests a systematic cause, not two independent card failures.
The refusal text says "the unit exists and the member is missing" without naming which file or package quarry searched, so the operator cannot tell which case applies.

What to do:
- Check whether quarry resolves `plan:` targets in `_test` external test packages and in test files newly created in the same batch.
- If the lookup is correct, make the `create-not-done` message name the files quarry searched so a genuinely missing test is told apart from a lookup miss.

Provenance: task `machine-handover`; notes `webster-refusal-record-batch.md`, `webster-refusal-record-batch-2.md`.
