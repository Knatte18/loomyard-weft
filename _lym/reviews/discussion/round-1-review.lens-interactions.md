```yaml
lens: interactions
outcome: reported
```

I worked the interactions lens: every pair of decisions that act on the same state, plus each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D3 (worktree maps written by the refresh, the reader picking worktree or hub map): held. The refresh writes only anchor maps, and D1's hash rule picks them only while they are current.
- D1 index × D3 / D11 (who writes the shared maps and the index): held. Neither writes the hub's shared maps, so the index stamps stay valid.
- D1 × D9 (survivor's other-package killers come from the reader): held.
- D2 × D7 (the run's changed functions used in triage lines): broken when there is no parent branch. See F2.
- D2 × D9 (changed lines from the same base): held. D9 needs a fresh map, which needs a clean package, so the worktree files equal HEAD for that package.
- D3 × D5 (`reach.json` matched against HEAD): held. The note says when it answered from older maps.
- D3 × D7 (triage on a red call, when the refresh never ran): held. D7's bound names stale maps.
- D3 × D9 (what counts as a fresh map): broken in the prime. See F3.
- D3 × D10 (order and budgets): held. D3 runs first whatever the list order, and the budgets are separate.
- D3 × D12 (the `reports` list order against D3's run-first rule): held.
- D5 × D10 (`mutation.json` matched "as D5 matches"): held.
- D6 × D8 (two forms of one verb): partly open. `--since` and `--branch` given with the `<file>#<Func>` form are not addressed. See F4.
- D7 × D8 (the D8 command appended to a D7 line): held. The line fits under the 300-byte line cap in the "reaches none" case, the only case that carries the command.
- D7 line cap × D7 acceptance (parse unchanged): broken. See F1.
- D10 × D12 (`mutation` without `reach` refused, against the filled template): held. The template lists no `mutation`, and the hub's base `gates` entries set no `reports`.
- D11 × D3 / D9 (`--base` with the refresh): held loosely. The refresh's changed set presumably follows `--base`.
- Constraints `PATTERN-gate-slot`, `PATTERN-lym-files` and `PATTERN-test-speed` × D1, D3, D6, D7, D9: held.

**Findings**

F1 — MEDIUM, design, decision-record.md D7 and its acceptance bullet "`gateslot.FailedTests` parses a report holding triage lines to the same failed tests as without them".
D7 has each triage line count toward the report's line cap, and its acceptance criterion promises an unchanged parse; the two cannot both hold.
Evidence:
- The cap is `failureReportLines = 80` across the whole report (internal/gatecli/failurereport.go:29).
- `renderFailureReport` cuts later runs' lines once that budget is spent and adds a `... ` cut line (failurereport.go:200-215).
- `FailedTests` returns ok=false on a cut report (internal/gateslot/runlog.go:293-296 doc; a `... ` line fails `reportRunTier`).
- Its callers act on that result: internal/loomshed/verifygate.go:248 and internal/landingshed/verifygate.go:309.

A report near the cap that triage lines push over it is cut, and those callers lose its named failed tests.
Fix: D7 must state how the triage lines avoid tipping the report into a cut, for example by being left out of the cap or dropped first, and say what bounds that.

F2 — MEDIUM, design, decision-record.md D7, with D2.
D2 diffs from the merge base with the parent branch, and D7 defines no case where there is no parent branch.
Without one, `parentMergeBase` returns found=false (internal/gatecli/scope.go:697-699), so the change set is empty or unreadable.
D7 would then print "reaches none of this run's changes" plus the D8 command for every mapped failing test, telling the fixer that a failure is not the run's own.
D8 covers its own no-parent case ("walks its current branch"); D7 and D3's no-judge cases do not.
Fix: D7, and D3's note, state that a call with no parent branch, or with a merge base that cannot be read, adds no triage line, or a line saying no change set exists.

F3 — LOW, consistency, decision-record.md D9's first bullet against D3 and D11.
D9 counts as fresh only a map D3 refreshed or a worktree map whose hashes match.
D3 refreshes nothing in the prime and answers there from the shared maps.
So a `mutation` step or a `lym gate mutate` call in the prime judges no package, even when the hub map's hashes match the files, and no decision says so.
Fix: D9 either accepts a matching hub map as fresh in the prime, or D9 and D11 state that a call in the prime judges nothing, with a note, as D3 does.

F4 — LOW, consistency, decision-record.md D6 and D8.
D6 refuses "both a function and `--test`, or neither", but nothing says what happens to `--since` or `--branch` given with the `<file>#<Func>` form.
Fix: D6 refuses `--since` and `--branch` without `--test`, naming the D8 form.

I made no tool calls; everything above comes from the inherited context.
