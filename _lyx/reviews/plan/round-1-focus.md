---
round: 1
focus:
  - "Fidelity, cards 3 and 7 against D1 and D6: does some card carry each docs and prompt-text deliverable the decision record names (orch role escalation item for the D1 goto, refusal-spec rows `Publish: verify failed` and `decision: task session`, the verb's help and package doc dropping the no-caller-identity sentence, `docs/overview.md`'s `decision add` verb line, the plan stencil replacement text including the no-parent case)?"
  - "Fidelity, card 2 against D4: does the card carry the three-point comparison (differs from both fork point and parent tip), the parent tip in the notice, and the four acceptance scenarios (synced parent change, task edit, unsynced parent change, task edit identical to the parent)?"
  - "Fidelity, card 3 against D1: does the card keep the no-goto reasons for timeout, unstartable verify, dirty tree and a failed record write, and carry the record writer's written-or-not report through to the Stuck reason?"
  - "Granularity, card 5: are `preMerge`, `newPreMerge` and `preMerge.clear` one reviewable unit with a helper-sized split into three Create targets, and does the card bundle `conflictSessionStopper` in `internal/loomcli` with the `internal/landingshed` pre-merge step where a separate card would review independently?"
  - "Test economy, cards 5 and 6: for `premerge_test.go`, `TestConflictSessionStopper` and the new `resume_test.go` / `start_midmerge_test.go` rows, does an existing table (`TestRunDriverSpawnAndWait_MidMerge_*`, `TestResumeVerb`, `TestDriverWaitMark`, the landingshed Publish and Finalize tests) already cover the behavior, so that a new test function or file is a finding that names the covering test?"
  - "Member glyphs and re-sign arrows, cards 3, 5 and 6: for each `Edit` target whose Intent changes parameters, results or receiver (the Publish gate methods, `landingDeps`, `refuseOverUnfinishedMerge`, `NewPublish` and `NewFinalize` if their signatures change), is the re-sign arrow present, and is any Go-file-only `Edit` target left where the Intent names symbols?"
  - "Attack-surface bound, cards 5, 6 and 7: for the abort that discards a parent-sourced parked merge-in whoever started it, the narrowed `merge_in_progress` guard and the new caller check on `--by`, does each card's `**Intent:**` state what it can now let through or discard and what bounds it (own parent source, stop before abort, unresolvable parent refuses)?"
  - "Card order and dependencies, cards 1, 4, 5, 6: does card 5 or card 6 use a symbol (`MidMergeState` fields, `ConflictRole`) that a later card creates, and does card 4's `Rename` leave every use of `conflictRole` covered?"
  - "Verify coverage, `## verify:` in the overview: does the section run the tests of every package the cards target, in the order the plan template states, with `llm` compiled and `tmux` compiled only?"
---

Seeded from the overview, the decision record and cards 4 to 6 read in full; the remaining cards were skimmed for type and `ImpactSummary` only.
No entry restates a gate-checked item, a missing `DependsOn`/`Produces`, or an `ImpactSummary` on a non-`Edit`/`Delete` card.
