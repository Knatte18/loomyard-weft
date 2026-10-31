```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changed code through the fix-code-correctness lens.

No findings.

I ran each changed branch with one concrete input and found no wrong result:
- **`decisionParent`, beam with a recorded shortname:** `NearestLiveOrch` returns a non-empty `Name`, because it falls back to the prime's orch when the beam's is not live. The new empty-`Name` guard therefore fires only for a hub with no shortname.
- **`decisionParent`, beam with no shortname:** the guard returns an error that reaches `decisionVerb` through the `deps.parent()` error branch. The refusal then carries the `lym beam start` wording instead of the "task session" text.
- **`decisionParent` with `record.project` false:** the path is unchanged.
- **`decisionReceivers`:** the new `runStrands` parameter is used for the target and for each pair. `preRun` passes `liveRunStrands`, so production behaviour is unchanged, and a compile error from a missed caller is ruled out because the tier0 and tier1 gates pass.
- **`preRun` seed refusal:** only the message text changed.
- **`ProjectRecordHeld` doc comment:** it matches `recordFreeProducers` and the Preflight / Record-Write behaviour the review describes.

Each added test fails with its defect in place:
- **`TestWithProjectDirective`:** a fill that is not skipped for a pair under no beam breaks the `loose` row's empty-marker and exact-values assertions. A fill that ignores the record breaks the `member` row.
- **`TestMemberPairs_ReturnsEveryLocalPairUnderTheBeam`:** the fixture has `member-with-pair` both on the board and forked from the beam. Removing the dedup in `memberPairs` makes that pair appear twice, and including `member-no-pair` also fails.
- **`TestDecisionReceivers_ReachABeamsMemberPairs`:** the project row requires the member strands and the discussion row forbids them.
- **`TestDecisionParent_ABeamsRecordIsItsOwnOrchs`:** without the guard, the no-shortname case returns `Name == ""` and no error. The `err != nil` assertion then fails.
- **`TestCommitRunRecord_ProjectRecordCommitsItsDirectory`:** if the project branch fell through to `commitDiscussion`, the project record would not be committed and the discussion record would be.
- **`wiring_beam_test.go`:**
  - The `ProjectDecisions` assertion reads an added decision that exists only at the project path.
  - The `DecisionRecordPath` assertions are per-recipe.
  - The arbiter prompt assertion would fail if `arbiterDepsFor` used the discussion path.
- **`TestRunRecordOf`:** `wantUnderLym` pins the `_lym`-relative form `preRun` derives.

Outside my lens: the burler geometry's `PatternRecordPath` at `wiring.go:528` has no assertion. F3 did not ask for one explicitly, so I did not treat it as a finding.

I made no further tool calls; my context already held every changed file.
