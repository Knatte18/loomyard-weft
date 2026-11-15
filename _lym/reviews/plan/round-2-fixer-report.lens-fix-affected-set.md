```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. For each finding whose premise is a set, I reran its search and checked that the fix covers every member.

Unchecked findings: none.

- **F1** (`CancelMarker` sites in the planner): `internal/dobbyengine` reads `run.CancelMarker` at `passes.go:124` (`planPass`) and `passes.go:203` (`drivePasses`' lift drop). The fixed card 11 keeps both. The busy probe at `passes.go:213` goes through `planPass`, so it inherits the gate. The other readers, `ended.go:37`, `runnotice.go:182` and `waitreminder.go:72`, are outside the planner and the review says they stay. All members are covered.
- **F2** (seams of `battenshed.FinalizePastMerge`): the seams are `restepped` and `parentHolds`. Card 5 now names the source of each, and the no-pair case calls neither. All members are covered.
- **F3** (card 5's test functions): there are two, `TestReadRun` and `TestReadRun_ReadsTheSameStateWithoutTheCache`, and the card now names the test package of each. All members are covered.
- **F4** (const-block members card 5 edits): those are `landRow` and `worktreeTeardownRow`, both now Edit glyphs. `runShedRow` is a Create target already. All members are covered.
- **F5** (verbs that render the terminal `Landed` refusal): the plan greps show card 10 returns `doneSlugRefusal` when a batten record stands, card 14 treats terminal as act, and card 7's `TestCancelRun` row asserts "the terminal wording" without quoting it. None of them quotes the old "`lym board list` to check it" text, so none goes stale. All members are covered.

no findings

Tool calls:
1. A grep of `CancelMarker` in `internal/dobbyengine` plus a grep of the plan cards for the Landed and `board list` wording. My context held only `passes.go` and three of the cards.
2. `sed` on card 7's lines 50–58 and a grep for terminal handling in cards 10–19. I needed the sibling cards that render or assert the Landed refusal, and my context did not have them.
