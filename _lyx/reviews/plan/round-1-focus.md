---
round: 1
focus:
  - "Card 15 (batten-decision-resume) reach: the ReadApproval -> ReadDecision field rename and the ChildApproval -> ChildDecision type rename are also referenced by internal/shedrecipe/entries_batten.go (requireSeam on InnerRun.ReadApproval), internal/shedrecipe/entries_batten_test.go and internal/battencli/wire_test.go, none of which appear in card 15's Edit list or any other card. Settle whether the plan as written leaves the post-merge build or tests broken."
  - "Card 12 (landingshed-publish-split) reach: internal/landingshed/publish_verify_test.go asserts Publish returns Awaiting after a passing verify, and no card edits it. Settle whether it must change once Publish returns Done at an open PR. Also check whether internal/loomrecipe/resume_test.go and sequence_test.go, which assert a halt at NamePublish, still hold under card 11's rerouting, since card 11 edits only shape_test, coverage_guard_test, recipe_test and fixture_test in that package."
  - "Docs fidelity (the decision record's docs-in-the-same-commit Constraint and D2's hand-made-fix statement) against card 16's scope: docs/overview.md's module-tree line 'landing's two general ShedProducers, Publish and Finalize', its sentence 'Publish returns Done on a valid approval ... before any sync', the batten sentence 'it does restart an approved awaiting one', and contracts/specs/loom-status-spec.md's 'awaiting is the planned PR-review hand-off at Publish'. Settle whether each is carried by some card."
  - "Card 9 (loomshed-pr-rework) re-entry and round numbering: step 2 must find a round-<N> directory committed at HEAD whose record.json carries the pending head_sha, and step 5 picks N from the working tree, but ReadCommitted reads a single anchor-relative file and the card does not say what the round directories are enumerated from. Settle whether the declared PRReworkDeps seams can implement D5's crash re-entry rule. Also check that step 4's ParsePlanFrom-over-ReadCommitted matches card 3's bare-file-name read contract and card 5's anchor-relative relPath contract."
  - "Card 6 (websterengine-rebaseline) against D6 and its acceptance criterion: the runlevel_test case seeds a state with batches 1-2 complete. Settle whether that state matches what a finished Webster run actually leaves (integration stage done, outcome.yaml and summary written), so the test proves the stale-outcome archive and the integration re-run D6 promises rather than an ordinary mid-run resume."
  - "ImpactSummary conclusions on cards 2, 12 and 13: settle whether each gives a blast-radius conclusion or restates Intent. For card 12 in particular, check whether it names who consumed Publish's Awaiting verdict (batten's awaiting watch, the loomrecipe and landingshed tests, the status spec)."
---

Every file named above was checked against the working tree before being promoted, and the checks returned these results.
`grep` for `ReadApproval`/`ChildApproval` returns three files outside card 15's list.
`publish_verify_test.go` asserts `shedengine.Awaiting` in five places.
The overview and status-spec sentences are quoted verbatim.

Excluded as out of scope for this rubric:
- Dependency or `Uses` completeness (derived, not authored).
- Missing `ImpactSummary` on the `Create`-only cards 4, 5 and 9 and on the `Prosa` card 16.
- Verify coverage: `## verify:` runs `go test ./...` and `go test -tags integration ./...`, and it only vets `smoke`.
- The rework row's extra `coverage.md` and `rejected_at` fields, which are implementation detail of D5's round record, not new scope.
