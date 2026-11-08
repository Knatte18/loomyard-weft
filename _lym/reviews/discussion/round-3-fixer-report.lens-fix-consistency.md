```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the parts of the decision record it touches or relies on.

**[MEDIUM] D5 Bound contradicts the new exit record**
- Changed text: D5, "That count reaches its caller through an exit record the verb writes as it exits: a record written through `internal/state` … holding the verb's process ID and the count."
- Problem: D5's Bound paragraph, which the fix did not touch, still says "Bound: it writes nothing but map records". The verb now also writes the exit record, so the Bound is false.
- Evidence: in the D5 Bound of `_lym/discussion/decision-record.md`, the sentence just before "In time, the wait for its gate slot is not bounded by the verb."

**[LOW] The exit record has a D6 acceptance bullet but no D5 bullet**
- Changed text: the D6 acceptance bullet "when D5's exit record names the refresh it waited on and a count of stale packages the cap left unbuilt above zero", and the new D5 exit-record sentence.
- Problem: the D5 decision now defines a record holding the process ID and the count. The D5 acceptance bullets, which the fix did not touch, still give only the exit listing: "lists each skipped package with its cause apart from the count the cap left unbuilt". No bullet requires the verb to write the record or say what it holds, so a test of D6 depends on behaviour that no D5 criterion pins.
- Evidence: the D5 acceptance bullets in the Acceptance criteria section.

**[LOW] The D7 not-judged criterion leaves out the spent budget**
- Changed text: D7 step 4, "Once it is spent the step starts no further build or test; each test it did not run, the tests of every package it did not build among them, is listed as not judged for the spent budget."
- Problem: the fix adds a new not-judged reason, and covers tests of packages left unbuilt. The D7 acceptance bullet that lists every not-judged reason does not include it. It reads "a `//testtiming:keep` test …, tests past `redundancy_max_tests`, a timed-out test, and a test that was skipped, failed or covered no block". No criterion says that a build does not start once the budget is spent.
- Evidence: the D7 acceptance bullet that begins "D7: a `//testtiming:keep` test with its keep reason".

Checked and consistent: the new `PATTERN-fabric-boundary` Constraints bullet agrees with D7 step 1, because `parentMergeBase` reads through `fabricengine.Open(...).MergeBase` (`internal/gatecli/scope.go:679-683`). D4's tracked-at-HEAD hash matches what D5 compares. D1's keep-scan move matches D7 step 2's new cross-reference. The rationale's "D5's grace" refers to D5's fixed grace.

Tool calls:
1. Grepped `internal/gateslot` for its non-tier holder wording, `internal/gatecli` for the `parentMergeBase` route and the lints' slot handling. My context did not show how those places take slots or whether the merge-base read goes through fabric.
2. Read `internal/gateslot/doc.go:58-72` to see whether the doc lists non-tier holders, which the new D7 step 4 text cites. My context had only the grep hits.
