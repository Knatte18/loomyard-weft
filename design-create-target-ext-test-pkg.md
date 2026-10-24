# webster record-batch refuses Create target when the test is declared in an external _test package

`create-target-ext-test-pkg` · Task · bug, webster, quarry

Depends on: [`webstercli-tests-ignore-tmp-git`](design-webstercli-tests-ignore-tmp-git.md)

Imported from [issue #861](https://github.com/Knatte18/loomyard/issues/861).

**What happened**

`lyx webster record-batch` failed two batches (`01-fabric-merge-in-at` and `33-landing-gate-at-fix-rows`) with a `create-not-done[blocking]` finding.
The card's Create target had the form `plan:internal/<pkg>#TestName`.
The check answered that the target "still does not resolve": `quarry answered not_found (the unit exists and the member is missing)`.
The refusal text itself says the test is present, declared as `internal/<pkg>_test#TestName` in an `internal/<pkg>/*_test.go` file.
Seen for `internal/planparser#TestIsTestFile` and `internal/landingshed#TestHandsBack`.

**Why it is abnormal**

The test exists and was written in the file the card asked for.
It sits in the Go external test package (`<pkg>_test`), which is the usual convention in this repository.
The Create-target resolution looks the member up only in the unit `internal/<pkg>`, so a member in the sibling `<pkg>_test` package is reported missing.
Both the card's work and the check are correct, so the batch is failed and the record is refused for a defect in the resolver, not in the task.
The refusal even names the real location, so the resolver has the information it needs.

**What to do**

Make the Create-target lookup for a `plan:` target also search the unit's external test package (`<unit>_test`), or accept a member quarry reports under that package.
Until then, a card creating a test in an external test package cannot pass `record-batch` and needs `lyx webster recover-batch`.

Provenance: task `loom-tail-review`; notes `webster-refusal-record-batch.md`, `webster-refusal-record-batch-2.md`.
